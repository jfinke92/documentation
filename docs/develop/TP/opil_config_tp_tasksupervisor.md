# TaskSupervisor

 The TaskSupervisor is a system monitor that provides information about the ongoing tasks, as well as the general status. The TaskSupervisor is responsible for publishing information about the running tasks as well as those that were stopped (not implemented yet).

The TaskSchedular can be divided into three divisions, as described below: 

* **TaskSchedular**: The TaskSchedular is responsible for creating and monitoring the TaskManager. The TaskSchedular denotes a collection of one or more TaskManager.
* **TaskManager**: The TaskManager is responsible for creating and monitoring tasks. A TaskManager denotes a collection of one or more tasks.
* **Task**: A task is the lowest part of the TaskSchedular and describes a transport order that is being executed. Each task includes some information about what will be transported, from where to which destination, how this task will be triggered (e.g. a Sensor Event like a button needs to be pushed) and if there is a follow up/child task.

 An example of this TaskSchedular is depicted in the following listing:
```
TaskSchedular
 ->TaskManager_1
	Task_1->Task_2->Task_3
 ->TaskManager_2
	Task_A->Task_B
 ->TaskManager_3
	Task_Foo

```

# Logistic Task Language

The Logistic Task Language *(LoTLan)* is a simple, but powerful approach to describe intralogistic materialflow logic. A materialflow processes is mainly a transportation task like the pickup *- go to position and get item -* and the delivery *- got to position and unload item*.

## Table of contents

- [TaskSupervisor](#tasksupervisor)
- [Logistic Task Language](#logistic-task-language)
  - [Table of contents](#table-of-contents)
  - [Introduction](#introduction)
    - [Use of Example](#use-of-example)
  - [Primitives](#primitives)
  - [Instances](#instances)
  - [Tasks](#tasks)
    - [Example simple Task](#example-simple-task)
    - [Example Trigger Task](#example-trigger-task)
    - [Example OnDone Task](#example-ondone-task)
  - [Comments](#comments)
  - [Full Example](#full-example)

___

## Introduction

The *LoTLan* consists of 3 different building blocks, that combined with each other describes such a process:

- Primitives
- Instances
- Tasks

A *Primitive* is an abstract model for a series of similar objects. It is the blueprint for the representation of real objects in software objects and describes attributes (properties) of the objects. Through the instantiation of such a *Primitive* a *Instance* of a concrete object is created. A *Task* then combines these *Instances* to a logical process flow.

### Use of Example

The following documentation of *LoTLan* utilizes the example of a production hall that has an area for storing goods *- the Warehouse -* and an area for the manufacturing *- the Production*. To reduce the complexity only one AGV out of a possible lager fleet is used.

<div style="text-align:center">

![Example introduction](./img/tl_introduction.png)

*Figure 1: Example floor plan with AGV and production & warehouse area*
</div>

This example shown in the figure above will be expanded in the course of time to explain the individual building blocks of the *LoTLan*.

## Primitives

A *Primitive* can contain multiple member variables, like palettes/storage places, multiple conveyer belts, or like sensors with the same capabilities. Therefore, a *Primitive* summarizes and specifies a set of *Instances*.

In the domain of logistics such a *Primitive* could be a position. Defining such a position is done via the *__template syntax__*:

```text
template Position
    type
    value
end
```

The *Primitive* *Position* has two member variables, a *type* and a *value*. These attributes can be later on accessed inside the instances.

**Syntax**: It is important that the attributes inside an *template - end* definition begin with a lowercase character. The name has to start with an uppercase character. Each value also needs to be prefixed with four spaces (or a `\t`).

## Instances

An *Instance* is the concrete object of a previously declared *Primitive*. Such set of *Instances* do not share any data other than the definition of their attributes.

As an example, two *Instances* of positions could be initiated out of the previously made *Primitive* (see [Primitives section](#Primitives)):

```text
Position goodsPallet
    type = "pallet"
    value = "productionArea_palletPlace"
end

Position warehousePos1
    type = "pallet"
    value = "warehouseArea_pos1"
end
```

The *Instance* *goodsPallet* has two member variables, a *type* and a *value*. The *type* attribute states *what item is located there* and the *value* the *logical name of this location*.

**Syntax**: The syntax of *Primitives* introduced before here is complemented by assigning values to the attributes. These values must be enclosed by `"`. Also, the name has to start with a lowercase character.

Speaking of the example introduced in the [introduction](#Logistic-Task-Language), those *Instances* each define a specific position inside the two areas.

<div style="text-align:center">

![Example instance](./img/tl_instances.png)

*Figure 2: Floor plan with Positions **goodsPallet** and **warehousePos1***
</div>

The figure shows those positions inside the two areas *Warehouse* and *Production*.

## Tasks

A *Task* orchestrates different *Instances* via operations to result in a logical process flow. Such a *Task* does not need to describe who is going to transport an item - it is important that the item will be transported.

Generally speaking a *Task* in *LoTLan* describes that a amount of items should be picked up at some positions and be delivered to another position. The *Task* can optionally be triggered by an event or by time and it can optionally issue a follow up *Task*:

```text
Task {name}
    Transport
    From        {Position_1}
    To          {Position_D}
    TriggeredBy {none|event}
    OnDone      {none|followUpTask}
end
```

To simplify this down in the following the simplest structure of a *Task* is build and later on extended with optional functionality.

### Example simple Task

In the simplest form a *Task* in *LoTLan* just describes that an item should be picked up at some position and be delivered to another position:

```text
Task TransportGoodsPallet
    Transport
    From        goodsPallet
    To          warehousePos1
end
```

In terms of the introduced example production hall this *Task* looks like depicted in the following figure.

<div style="text-align:center">

![Example task](./img/tl_tasks.png)

*Figure 3: Floor plan with Task **TransportGoodsPallet***
</div>

This *Task* *TransportGoodsPallet* could be done by an AGV, that picks up a pallet **from** *goodsPallet* inside the production area and delivers it **to** the *warehousePos1* in the warehouse area.

### Example Trigger Task

A *Task* can be extended with a *TriggeredBy* statement that activates that *Task* if the case occurs. This statement can be an event like a button press or be something simple as a specific time:

```text
Task TransportGoodsPallet_2
    Transport
    From        goodsPallet
    To          warehousePos1
    TriggeredBy buttonPallet.value == True
end
```

In this example, the *Task* *TransportGoodsPallet_2* will be triggered by a sensor event, when a button has been pushed and the value is equal (*== True*).

In terms of the introduced example production hall this *Task* looks like depicted in the following figure.

<div style="text-align:center">

![Example trigger task](./img/tl_tasks_triggeredby.png)

*Figure 3: Floor plan with Task **TransportGoodsPallet_2***
</div>

This *Task* *TransportGoodsPallet_2* could be done by an AGV, that picks up a pallet **from** *goodsPallet* inside the production area and delivers it **to** the *warehousePos1* in the warehouse area, when the button *buttonPallet* is pressed.

### Example OnDone Task

A *Task* can be extended with a *OnDone* statement that activates another *Task* when the original one has ended:

```text
Task Refill
    Transport
    From        warehousePos1
    To          goodsPallet
end

Task TransportGoodsPallet_3
    Transport
    From        goodsPallet
    To          warehousePos1
    TriggeredBy buttonPallet.value == True
    OnDone      Refill
end
```

In this example another *Task* is introduced. This *Task* *Refill* is the same transport as the formerly introduced *TransportGoodsPallet*, just the other way around. On the other hand, *TransportGoodsPallet_3* here shows now the *OnDone* statement that points to *Refill* an runs that *Task* if done. That means a concatenation of *Tasks* is allowed. Exploiting this behaviour infinite *Tasks* can be managed by pointing to each other. So *Refill* could also point to *TransportGoodsPallet_3* in a *OnDone* statement.

In terms of the introduced example production hall this *Task* looks like depicted in the following figure.

<div style="text-align:center">

![Example on done task](./img/tl_tasks_ondone.png)

*Figure 3: Floor plan with Task **TransportGoodsPallet_3** & **Refill***
</div>

This *Task* *TransportGoodsPallet_3* could be done by an AGV, that picks up a pallet **from** *goodsPallet* inside the production area and delivers it **to** the *warehousePos1* in the warehouse area, when the button *buttonPallet* is pressed. After that the AGV executes the *Task* *Refill* and so, it picks up **from** the *warehousePos1* and delivers it **to** the *goodsPallet* position.

## Comments

A comment starts with a hash character (`#`) that is not part of a string literal, and ends at the end of the physical line. That means a comment can appear on its own or at the end of a statement. In-line comments are not supported.

```text
###
# This task shows the usage of comments in LoTLan
###
Task TransportPalettTask
    Transport
    From        palett1, palett2, palett3  # All Pallets!
    To          warehousePos1
    TriggeredBy warehousePos1.value == True  # More comments
    OnDone      Refill
end
```

This example shows a mimicked multi-line comment that consists of three `#` that are joined together.

## Full Example

```text

###
# Defining a Primitive Position with the two attributes type and value
###
template Position
    type
    value
end

###
# Initiation of the two Positions goodsPallet and warehousePos1
###
Position goodsPallet  # Using the Primitive Position
    type = "pallet"
    value = "productionArea_palletPlace"
end

Position warehousePos1
    type = "pallet"
    value = "warehouseArea_pos1"
end

###
# Creation of a Task that transports from goodsPallet to warehousePos1
###
Task TransportGoodsPallet
    Transport
    From        goodsPallet
    To          warehousePos1
end

###
# Creation of a Task that is triggered if buttonPallet is pressed
###
Task TransportGoodsPallet_2
    Transport
    From        goodsPallet
    To          warehousePos1
    TriggeredBy buttonPallet.value == True  # buttonPallet is a Sensor
end

###
# Creation of a Task that will call Refill when done
###
Task Refill
    Transport
    From        warehousePos1
    To          goodsPallet
end

Task TransportGoodsPallet_3
    Transport
    From        goodsPallet
    To          warehousePos1
    TriggeredBy buttonPallet.value == True
    OnDone      Refill  # If this Task is done, call Refill
end

###
# A infinite loop described by two Tasks that point on each other
###
Task Refill
    Transport
    From        warehousePos1
    To          goodsPallet
    OnDone      TransportGoodsPallet_3
end

Task TransportGoodsPallet_3
    Transport
    From        goodsPallet
    To          warehousePos1
    TriggeredBy buttonPallet.value == True
    OnDone      Refill
end
```
  