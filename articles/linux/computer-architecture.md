---
layout: default
title: Computer Architecture
parent: Linux Internals
grand_parent: Articles
nav_order: 1
description: "From transistors to a working CPU: logic gates, adders, latches, registers, RAM, and the fetch-decode-execute cycle."
redirect_from:
  - /linux-internals/computer-architecture/
  - /linux-internals/computer-architecture.html
---

# Computer Architecture

How a CPU is actually built, bottom-up: transistors become logic gates, logic
gates become adders and decoders, and those combine into memory (latches,
registers, RAM) and a control unit that fetches, decodes, and executes
instructions.

## Part 1 — Transistors, Logic Gates & the ALU

**Understanding Transistors and Logic Gates**

Transistors are very simple components; they are basically electronic switches. When we apply current to one of its terminals, a transistor lets electricity pass through. But how can this simple behavior make computers possible? In today's video, we are going to learn how transistors can be used to achieve more complex tasks like doing math and even interpreting instructions.

A single transistor typically has three terminals: a collector, an emitter, and a base. In a circuit, it can act as an insulator, preventing electricity from flowing between the collector and emitter terminals.
![1750172983097](/assets/images/notes/computer_achitecture_part_1/1750172983097.png)

But if we apply a small current to the base terminal, it acts as a conductor, allowing electricity to flow. Essentially, a transistor can be seen as a switch, but instead of mechanical movement, it operates by using electrical signals.
![1750174244574](/assets/images/notes/computer_achitecture_part_1/1750174244574.png)

In this example circuit, we are using a transistor to turn on and off an LED. By utilizing the base terminal as an input and the emitter terminal as an output signal, we can mimic the input signal with a switch and visually represent the output signal with the LED.
![1750174425671](/assets/images/notes/computer_achitecture_part_1/1750174425671.png)
![1750174458663](/assets/images/notes/computer_achitecture_part_1/1750174458663.png)
![1750174477806](/assets/images/notes/computer_achitecture_part_1/1750174477806.png)

Let's dub this a simple gate. In a simple gate, when the input is zero, the output is zero. When the input is one, the output is one.
![1750174498793](/assets/images/notes/computer_achitecture_part_1/1750174498793.png)
![1750174508000](/assets/images/notes/computer_achitecture_part_1/1750174508000.png)

Now let's tweak things a bit here. When the input is zero, the LED is on due to the way it is arranged within the circuit. But pay attention to this: setting the input to one causes the LED to turn off, indicating to us that the output is a zero. This setup is commonly referred to as an inverter or a NOT gate.
![1750174553266](/assets/images/notes/computer_achitecture_part_1/1750174553266.png)
![1750174577068](/assets/images/notes/computer_achitecture_part_1/1750174577068.png)

This may seem a bit confusing. This video I found on Twitter is a perfect example of this effect, and if you want more details, you can watch this video by Ben Eer, where he explains all this using real components.
![1750174648547](/assets/images/notes/computer_achitecture_part_1/1750174648547.png)
![1750174676590](/assets/images/notes/computer_achitecture_part_1/1750174676590.png)

We're not limited to just one transistor. By using multiple transistors, we can achieve more complex behavior. If two transistors are connected in series, when both inputs are zero, the output will be zero because both transistors act as insulators.
Similarly, if either input is zero, the output will be zero. The only way to obtain a one in the output is by having both transistors act as conductors, which happens when both inputs are set to one.
![1750174740391](/assets/images/notes/computer_achitecture_part_1/1750174740391.png)
![1750174747081](/assets/images/notes/computer_achitecture_part_1/1750174747081.png)
![1750174754992](/assets/images/notes/computer_achitecture_part_1/1750174754992.png)
![1750174766677](/assets/images/notes/computer_achitecture_part_1/1750174766677.png)

This is where we begin to apply a powerful concept called abstraction. Instead of focusing on the individual transistors in the circuit, we can abstract this into a white box—a box that consistently outputs a specific value based on two given inputs. This is known as an AND gate. An AND gate outputs a value of one if and only if both inputs are one; otherwise, the output will be zero.
![1750174830021](/assets/images/notes/computer_achitecture_part_1/1750174830021.png)
![1750174842491](/assets/images/notes/computer_achitecture_part_1/1750174842491.png)

If we connect the transistors in parallel, when both transistors act as insulators, electricity cannot flow. But in this setup, having any transistor acting as a conductor is enough to allow electricity to flow. So in this arrangement, to get an output of one, it is not strictly necessary to set both inputs to one. Once more, we can abstract this circuit into a box known as an OR gate. An OR gate outputs a value of zero if and only if both inputs are zero; otherwise, it outputs a value of one.
![1750174920397](/assets/images/notes/computer_achitecture_part_1/1750174920397.png)
![1750174947603](/assets/images/notes/computer_achitecture_part_1/1750174947603.png)

These circuits, known as logic gates, are so fundamental that instead of representing them with boxes, each one is assigned a dedicated symbol. Notice that since inputs and outputs are electrical signals, we can connect the output of one logic gate to the input of another. This allows us to combine logic gates to achieve even more complex behavior. For example, if we desire this particular behavior, we can combine logic gates accordingly to achieve it. This is where we begin to see the power of abstractions. While this setup is ultimately composed of transistors, it's much simpler to understand what's happening by thinking in terms of logic gates.
![1750175013262](/assets/images/notes/computer_achitecture_part_1/1750175013262.png)
![1750175022369](/assets/images/notes/computer_achitecture_part_1/1750175022369.png)

This circuit is known as an XOR gate, and it is also very important in computer science, so it has its own symbol.
![1750175128974](/assets/images/notes/computer_achitecture_part_1/1750175128974.png)
![1750175147212](/assets/images/notes/computer_achitecture_part_1/1750175147212.png)
![1750175167328](/assets/images/notes/computer_achitecture_part_1/1750175167328.png)

 And now that we understand what logic gates are, let's use them to create more useful things. Let's start with adders. Adding binary numbers is actually quite simple. In fact, binary addition works just like decimal addition: 0 + 0 = 0, 0 + 1 = 1, 1 + 0 = 1, and 1 + 1 = 2. However, the value two cannot be represented with a single binary digit. When this occurs, we say the addition has overflowed, meaning we require an additional bit to represent the value.
![1750175257766](/assets/images/notes/computer_achitecture_part_1/1750175257766.png)

So what we're aiming for is a circuit that takes two input values and produces two outputs: the sum and the carry. Let's break down the sum. But before, We require a circuit that outputs zero when both inputs are the same and one when they differ. Does this sound familiar? It's precisely what an XOR gate accomplishes, so we've already solved half of the puzzle.
![1750175412504](/assets/images/notes/computer_achitecture_part_1/1750175412504.png)
![1750175427012](/assets/images/notes/computer_achitecture_part_1/1750175427012.png)
![1750175447898](/assets/images/notes/computer_achitecture_part_1/1750175447898.png)

For the carry output, we need a circuit that outputs the value one only when both inputs are one. This matches the behavior of an AND gate, and there we have it! We've crafted a circuit capable of adding two single-digit binary numbers.
![1750175495520](/assets/images/notes/computer_achitecture_part_1/1750175495520.png)

Okay, but what about multi-digit binary addition? Well, let's try. Since we have to add numbers in each row, using one adder for each row seems logical. It works well for the first row, but when we move to the second row, there's an issue. We need to consider that the previous row might have created a carry, but because the adder circuit only accepts two inputs, it doesn't account for this carry. Because of this limitation, this circuit is known as a half adder, and it is not useful in this scenario.
![1750175671844](/assets/images/notes/computer_achitecture_part_1/1750175671844.png)
![1750175691976](/assets/images/notes/computer_achitecture_part_1/1750175691976.png)

What we need is a full adder—a circuit that can handle not only the two bits being added but also take into account the carry from a previous addition. Here's the circuit we're working towards. Think of it as a circuit capable of adding three single-digit binary numbers.
![1750175810815](/assets/images/notes/computer_achitecture_part_1/1750175810815.png)
![1750175857317](/assets/images/notes/computer_achitecture_part_1/1750175857317.png)
![1750175872312](/assets/images/notes/computer_achitecture_part_1/1750175872312.png)

To simplify matters, we can encapsulate this in a box, which we'll call a full adder. With full adders, we can now add multi-digit binary numbers. The carry output of each full adder feeds directly into the carry input of the next full adder, as shown here in this animation.
![1750175923572](/assets/images/notes/computer_achitecture_part_1/1750175923572.png)

To add binary numbers of n digits in one go, n full adders are needed. For example, if we aim to add two 8-bit numbers, we'll need eight full adders.
![1750175952509](/assets/images/notes/computer_achitecture_part_1/1750175952509.png)

![1750176494031](/assets/images/notes/computer_achitecture_part_1/1750176494031.png)
**Remember that values are represented here using electrical signals. Since electricity moves at incredible speed, once we alter the input, the output changes almost instantly, and this is what makes transistors ideal for this job. While logic gates can be crafted from other components like relays and even fancy 3D printed parts powered by weird stuff like marbles or even water, none of this matches the speed and compactness of transistors.**

Before things become overly complex, we can one more time package all of this functionality into a special component known as an 8-bit adder. An 8-bit adder takes two 8-bit numbers as input and produces the sum of the inputs as another 8-bit number. It also provides an overflow signal, which is essentially the carry-out output of the last full adder. This overflow signal is crucial because it informs us whether the storage capacity being utilized is adequate to represent the result of the operation. In this example, both inputs are one byte long, but if we attempt to store the output in just one byte, we will miss information, resulting in an incorrect value. By monitoring the overflow output, we can recognize the need for an extra byte to accurately store the output.
![1750176588132](/assets/images/notes/computer_achitecture_part_1/1750176588132.png)
![1750176607623](/assets/images/notes/computer_achitecture_part_1/1750176607623.png)
![1750176620284](/assets/images/notes/computer_achitecture_part_1/1750176620284.png)
![1750176634081](/assets/images/notes/computer_achitecture_part_1/1750176634081.png)

Believe it or not, neglecting to manage operation overflows can lead to undefined behavior, sometimes with severe consequences, like this rocket accident in 1996. And we can continue to move to higher levels of abstraction. If we want to make a circuit that increments an input value, we can simply use an adder and set the second input to always be one. We've only talked about adders in detail, but in the end, it's all about logic gates. With them, we can make more useful things like a full subtractor that then can be used to build an 8-bit subtractor, and from there, we could keep going and make even more complicated stuff.

As you may have guessed, inside the CPU, there's a special component that houses all these circuits. For now, let's call it our mysterious component. The question we want to answer now is: when a CPU reads an instruction, how does it identify which one of these circuits corresponds to that specific instruction? How does the computer discern adding numbers upon encountering a particular instruction and subtracting them upon encountering another? I mean, some instructions aren't even arithmetic operations.
![1750176751695](/assets/images/notes/computer_achitecture_part_1/1750176751695.png)
![1750177883459](/assets/images/notes/computer_achitecture_part_1/1750177883459.png)
![1750177894180](/assets/images/notes/computer_achitecture_part_1/1750177894180.png)

**The last type of component we are covering today are binary decoders.** **Let's take a look at this circuit and examine the output for every input combination. The first thing we can notice here is that each combination triggers a specific output to activate while deactivating all other outputs. Another way to see it is that the circuit receives the binary number that corresponds to the position of the output we wish to activate. This is what binary decoders do:** when it receives an input, one and only one output has the value of one, with all others outputting the value of zero. If we have a decoder with three inputs, we can control which of eight outputs turns on; with four inputs, we can control 16 outputs, and so on.
![1750177936001](/assets/images/notes/computer_achitecture_part_1/1750177936001.png)
![1750177976774](/assets/images/notes/computer_achitecture_part_1/1750177976774.png)
![1750178004733](/assets/images/notes/computer_achitecture_part_1/1750178004733.png)
![1750178016707](/assets/images/notes/computer_achitecture_part_1/1750178016707.png)

This is huge because it means that we are also capable of creating circuits that can select among multiple options. **Now remember that assembly code is just a human-friendly representation of machine code—the actual code consisting of ones and zeros that computers comprehend. Not all instructions are arithmetic operations; some of them are instructions dedicated to fetching and writing data to memory and other stuff like that.**
![1750178058554](/assets/images/notes/computer_achitecture_part_1/1750178058554.png)

***Let's imagine a very basic architecture where if the first two bits of an instruction are zeros, the computer interprets them as arithmetic operation instructions***.
![1750178088035](/assets/images/notes/computer_achitecture_part_1/1750178088035.png)

In this example, to verify this, we could employ NOR gates. Given the scope of this video, at the moment we are not concerned with instructions that signify something else, but at least we know how to identify between them. In this example architecture, the third and fourth bits will determine the type of arithmetic operation to be executed. **For instance, if those bits are 00, it means addition; if 01, it represents subtraction, and so forth. This is commonly known as an opcode. Each opcode is associated with one and only one kind of arithmetic operation.**
![1750178167826](/assets/images/notes/computer_achitecture_part_1/1750178167826.png)
![1750178336364](/assets/images/notes/computer_achitecture_part_1/1750178336364.png)

**When the CPU has determined that the current instruction is an arithmetic operation, our mysterious component receives this opcode and internally links those two bits to a decoder, which is used to identify the desired internal operation. The straightforward approach here would be by allowing all circuits to receive the inputs and generate their respective outputs, but the outputs of the decoder are interconnected in a way that allows only the output of the selected operation to pass through this component. Keep in mind that there are more efficient ways to do this, but here we focus on simplicity.**
![1750178373459](/assets/images/notes/computer_achitecture_part_1/1750178373459.png)
![1750178416710](/assets/images/notes/computer_achitecture_part_1/1750178416710.png)
![1750178444979](/assets/images/notes/computer_achitecture_part_1/1750178444979.png)
![1750178469357](/assets/images/notes/computer_achitecture_part_1/1750178469357.png)
**Our mysterious component can also be enclosed within a box. This is a rudimentary and somewhat incomplete version of something known as an arithmetic logic unit.** We'll talk about this component in more detail in a future episode where we discuss how CPUs execute instructions. But beforehand, **an arithmetic logic unit takes input values and an opcode that tells the internal circuitry what arithmetic operation to perform between those values. It then produces the result of the specified operation along with additional information such as whether the result is negative, zero, or if it has overflowed.**
![1750178484219](/assets/images/notes/computer_achitecture_part_1/1750178484219.png)

## Part 2 — Memory: Latches, Registers & RAM

learned how transistors can be used to create more complex components such as adders, decoders, and even a basic arithmetic logic unit, but we didn't cover how computers remember information. So, hold on to your seat, because in this video we are going to create memory from transistors.

![1750348679812](/assets/images/notes/computer_achitecture_part_2/1750348679812.png)
![1750348778995](/assets/images/notes/computer_achitecture_part_2/1750348778995.png)

Let's examine the behavior of an OR gate in this situation. We establish a default state where everything is off. Remember, the output of an OR gate is zero if and only if both inputs are zero. When we set the first input of the gate to one, the output changes accordingly, and for a brief moment measured in nanoseconds, the circuit looks like this.
![1750349054266](/assets/images/notes/computer_achitecture_part_2/1750349054266.png)

But because electricity moves extremely quickly, this output value propagates back to the input almost instantly, changing the second input to one. Now, with the second input of the OR gate set to one, the output remains one regardless of the value we set for the first input, as long as the transistors are powered, of course. So, in other words, this circuit is remembering that the output was set to one at some point.
![1750349092247](/assets/images/notes/computer_achitecture_part_2/1750349092247.png)

![1750349132624](/assets/images/notes/computer_achitecture_part_2/1750349132624.png)

With the AND gate, we can get something similar. Recall that an AND gate outputs one only when both inputs are one.
![1750349171995](/assets/images/notes/computer_achitecture_part_2/1750349171995.png)

If we start from this state and then change the first input to zero, the output will become zero.
![1750349217544](/assets/images/notes/computer_achitecture_part_2/1750349217544.png)
Again, for a brief duration, the circuit resembles this configuration, but the output immediately propagates back to the second input. Now, with one of its inputs permanently set to zero, the gate will consistently output zero regardless of the value we set in the other input. Thus, in a sense, we could say the circuit remembers that its output value was set to zero.
![1750349239253](/assets/images/notes/computer_achitecture_part_2/1750349239253.png)

![1750349251558](/assets/images/notes/computer_achitecture_part_2/1750349251558.png)

Now we have two circuits: one capable of remembering a zero and the other capable of remembering a one. However, individually, they are not particularly useful, because what we need is a circuit that can remember either a one or a zero, not just one of them. Intuitively, our next step is to attempt combining both circuits and observe the outcome.
![1750349329680](/assets/images/notes/computer_achitecture_part_2/1750349329680.png)

![1750349338688](/assets/images/notes/computer_achitecture_part_2/1750349338688.png)

By default, the circuit looks like this. The value held by the circuit depends ultimately on this AND gate. In order to get an output of one, we can set both inputs to one, and since the output goes back to the OR gate, we can reset the first input back to zero without changing the output—but not the second input, because it goes directly to the AND gate.
![1750349389515](/assets/images/notes/computer_achitecture_part_2/1750349389515.png)

![1750349408772](/assets/images/notes/computer_achitecture_part_2/1750349408772.png)

![1750349467286](/assets/images/notes/computer_achitecture_part_2/1750349467286.png)

Let's add an inverter to the second input and see what happens.
![1750349553850](/assets/images/notes/computer_achitecture_part_2/1750349553850.png)

![1750349611776](/assets/images/notes/computer_achitecture_part_2/1750349611776.png)

By default, the circuit looks like this. Now, with this arrangement, if we set the first input to one, the output of the AND gate will indeed be one, because this inverter ensures that the other input of the AND gate is one even though the second input of the circuit is zero. The output of the AND gate, directly connected to one of the inputs of the OR gate, ensures that setting the first input back to zero will still retain the value of one in the output.
![1750349663257](/assets/images/notes/computer_achitecture_part_2/1750349663257.png)

![1750349704988](/assets/images/notes/computer_achitecture_part_2/1750349704988.png)

Similarly, if we set the second input to one while keeping the first input at zero, the inverter will cause the AND gate to output zero, and once again, the output feeding back to the input of the OR gate will make the circuit maintain a value of zero. Because even if we set the second input back to zero, the AND gate needs its two inputs to be one in order to output one, so it will retain a zero in the output instead.
![1750349859290](/assets/images/notes/computer_achitecture_part_2/1750349859290.png)

In other words, this circuit can remember one bit of information. Whenever we need to determine the current value held in the circuit, all we need to do is read the value from this wire. And there we have it—we've successfully created memory. We can encapsulate this circuit within a box that we'll refer to as an AND-OR latch. The first input allows us to set the stored value to one, while the second input allows us to reset the stored value back to zero.
![1750349946664](/assets/images/notes/computer_achitecture_part_2/1750349946664.png)

![1750349980693](/assets/images/notes/computer_achitecture_part_2/1750349980693.png)

![1750350064136](/assets/images/notes/computer_achitecture_part_2/1750350064136.png)

![1750350074316](/assets/images/notes/computer_achitecture_part_2/1750350074316.png)

![1750350090214](/assets/images/notes/computer_achitecture_part_2/1750350090214.png)

Unfortunately, with this setup, things get a little weird when we set both inputs to one at the same time. I encourage you to explore what happens when you attempt to do so. Short answer: it doesn't make sense to try to set the stored bit to one while simultaneously resetting it back to zero. If we desire a more intuitive mechanism, we require additional circuitry that enables us to specify which value we want to store using a single input. After all, a single wire can transmit either a one or a zero, right?
![1750350121525](/assets/images/notes/computer_achitecture_part_2/1750350121525.png)

Well, yes, but there's a complication. As soon as the input changes, the output value changes as well, so it no longer fulfills the function of memory. Instead, it simply outputs the current input value.
![1750350184166](/assets/images/notes/computer_achitecture_part_2/1750350184166.png)

![1750350193552](/assets/images/notes/computer_achitecture_part_2/1750350193552.png)

To address this issue, we introduce another input that we can activate whenever we wish to set the value being remembered. We'll name this new input as WR enable. With the addition of a few extra logic gates, we can achieve a more useful behavior.
![1750350255046](/assets/images/notes/computer_achitecture_part_2/1750350255046.png)

![1750350268915](/assets/images/notes/computer_achitecture_part_2/1750350268915.png)

When the write enable input is set to zero, the value in the data signal is completely ignored and nothing will happen. However, when the write enable input is set to one, the circuit will retain whatever value is being passed to the data input—in this example, one—and the value will persist even if we set the inputs back to zero later. If there comes a time when we need to overwrite the value stored in the circuit, all we have to do is activate the write enable signal while inputting the desired new value we wish to store—in this case, zero. As long as the transistors that make up this gate are connected to the current, the value will be remembered.
![1750350348677](/assets/images/notes/computer_achitecture_part_2/1750350348677.png)

![1750350375035](/assets/images/notes/computer_achitecture_part_2/1750350375035.png)

![1750350388563](/assets/images/notes/computer_achitecture_part_2/1750350388563.png)

![1750350407740](/assets/images/notes/computer_achitecture_part_2/1750350407740.png)

![1750350425517](/assets/images/notes/computer_achitecture_part_2/1750350425517.png)

One more time, we can abstract this circuit into a component known as a gated latch. And before continuing, I just want to clarify that there are several other ways to create latches; what I'm showing you here is not the only way to create circuits capable of remembering information. A gated latch is capable of storing one bit of information. If its write enable input is set to zero, no action will occur, but if the WR enable input is set to one, it will retain whatever bit of information is passed through the data input.
![1750350542844](/assets/images/notes/computer_achitecture_part_2/1750350542844.png)

With just a single bit of information, we can represent basic data like one or zero, true or false, or simple distinctions like black or white—but that's it, nothing too sophisticated. If we aim to represent a broader range of numbers or a larger set of colors, a single bit becomes insufficient—well, not really useful. But we can use multiple latches to expand the range of values we can store. For example, to store one byte of information, we can employ eight latches and combine all of their write enable inputs into a single input that we can use whenever we want to override all of the individual latches at the same time. A collection of latches interconnected in this way is known as a register—in this example, an 8-bit register.
![1750350816409](/assets/images/notes/computer_achitecture_part_2/1750350816409.png)
![1750350844125](/assets/images/notes/computer_achitecture_part_2/1750350844125.png)
![1750350856017](/assets/images/notes/computer_achitecture_part_2/1750350856017.png)

![1750350895610](/assets/images/notes/computer_achitecture_part_2/1750350895610.png)
![1750350902595](/assets/images/notes/computer_achitecture_part_2/1750350902595.png)
![1750350923418](/assets/images/notes/computer_achitecture_part_2/1750350923418.png)

The primary challenge with registers is their demand for numerous wires. The quantity of latches within a register determines its length, which can vary. Registers come in different sizes. However, larger registers imply an increase in both inputs and outputs. For instance, achieving a 1 GB storage capacity would require a big register with millions of wires for data inputs and outputs. If we want to remember bigger amounts of data, we need another solution.

Instead of arranging latches in a linear array, we'll organize them into a 4x4 matrix—totaling 16 latches. Our task now is to establish a method for identifying each one. To accomplish this, we'll utilize four wires for columns and four for rows. By activating a single wire in both the column and row, we can pinpoint a unique latch in the matrix. If we think about it, this mirrors the process of locating addresses on a map, where streets and avenues are typically numbered, so we can say each latch in the matrix has an address.
![1750351042831](/assets/images/notes/computer_achitecture_part_2/1750351042831.png)

Next, we need special circuitry to activate these wires in the described way. Fortunately, we've previously explored decoders. A decoder, when given an input, activates only one of its outputs. For instance, inputting 0 activates output zero; inputting 01 activates output one, and so forth. By employing one decoder for rows and one for columns, we can input an address to select a specific latch in the matrix. For example, inputting 000000 activates the first output of each decoder; inputting 00001 activates the first output of the column decoder and the second output of the row decoder, thereby selecting a different latch in the matrix.
![1750351103155](/assets/images/notes/computer_achitecture_part_2/1750351103155.png)
![1750351119272](/assets/images/notes/computer_achitecture_part_2/1750351119272.png)

Each latch possesses its unique address, which gives rise to the term memory address. Now that we can identify each latch in the matrix by its address, our next step is to devise a method for both reading and writing its value using the fewest wires possible. We can start by adding a few extra components to each latch so we can use a single wire both as input and output. To do this, we create a new input to let us toggle between reading and writing to the latch. Activating the read enable signal configures the data line as an output.
![1750351233297](/assets/images/notes/computer_achitecture_part_2/1750351233297.png)
![1750351316833](/assets/images/notes/computer_achitecture_part_2/1750351316833.png)

Now, someone might ask in the comments about the output signal feeding back into the input of the latch during reading. This is not a problem, since the write enable signal is inactive, thus disregarding any input. When the read enable input is off, the transistor prevents the signal from reaching the output, allowing the data line to serve as an input. With the data line functioning as an input, we can set the write enable data to one, overwriting the current value stored in the latch. All latches within the matrix can be modified in this manner.
![1750351448958](/assets/images/notes/computer_achitecture_part_2/1750351448958.png)

Having the required number of data wires—instead of 32 wires (16 inputs and 16 outputs), we only need 16 wires that can be toggled using the write enable and read enable inputs. But we can take this further by connecting the data lines of all latches to a single general data line, the write enable input of each latch to a single write enable line, and the read enable input of all latches to a single read enable line. We can set the value of all latches in the matrix without employing 16 data input wires and 16 write enable wires. Making all the latches remember the same value is not the behavior we want, though—we need each latch to remember its own value.
![1750351619599](/assets/images/notes/computer_achitecture_part_2/1750351619599.png)

![1750351646496](/assets/images/notes/computer_achitecture_part_2/1750351646496.png)

![1750351693383](/assets/images/notes/computer_achitecture_part_2/1750351693383.png)
Fortunately, we are almost there, because we already know how to use decoders to activate the cables intersecting precisely where each latch resides. We can use the trusty old AND gate to make some clever modifications to ensure that the write enable and read enable signals only reach the intended latch specified in the address input. As soon as the address input changes, the AND gate obstructs any passage of input signals through this point. So, in this example, even though the data input of the latch is set to zero, the latch will retain the value one in its memory because its write enable input is zero.

When all of the latches are connected in the same way, we get this—beautiful, isn't it? Let's see this circuit in action. Setting the write enable input to one allows the signal to traverse the matrix, yet it won't reach any specific latch due to the AND gates blocking its path. For the signal to target a particular latch, an address must be provided, prompting the decoders to activate one wire each for the columns and rows. The sole AND gate producing an output of one, thereby enabling the signal to flow to the latch, is where the active outputs of the decoders intersect. That's why this latch just changed its value from one to zero, as dictated by the value on the data line now utilized as input.

To revert the value of this latch back to one, we simply maintain this configuration while inputting one into the data line. If the address input changes, the output of the decoders will change, causing the activated outputs to intercept at a different location, and the write enable signal will exclusively target that specific latch, thereby overwriting its value with the data passed through the data line.

If we intend to read the value of a specific latch, we supply an address and activate the read enable signal. This will make the signal traverse the matrix but exclusively target the latch specified by the address, making the data line act as an output. And this is how we read the value any specific latch in the matrix is currently remembering.
![1750351798051](/assets/images/notes/computer_achitecture_part_2/1750351798051.png)
It's noteworthy that the value of the latch being read is propagated throughout the matrix, directly reaching the input of all other latches. Since none of the latches in the circuit have their write enable input activated, this value remains disregarded, so the value contained in the other latches won't be overwritten.

Okay, what you see here is actually an oversimplification, and even so, this is starting to get too complicated. I might have gone too far this time, so let's abstract all of this. With the address input, we can select any of the internal latches; by using the read enable signal, we make the data line output the current value of the selected latch. Conversely, if we use the write enable signal, we can override the value in the selected latch using the data line as input.
![1750352336455](/assets/images/notes/computer_achitecture_part_2/1750352336455.png)

Now, we are still dealing with single bits. You probably already know that the minimum data computers handle are bytes. To store bytes instead of bits, we can use eight matrices. All these matrices will share the same address input, allowing us to select corresponding latches across the matrices using the same address. The read enable and write enable inputs should also be shared, as we want to read or write an entire byte, not just individual bits. Each matrix will have its own data line, providing access to any bit within the byte.
![1750352384839](/assets/images/notes/computer_achitecture_part_2/1750352384839.png)
![1750352420685](/assets/images/notes/computer_achitecture_part_2/1750352420685.png)

For example, to write a byte to memory, we input all the bits that make up the byte through the data lines and then activate the write enable signal. This action causes each bit of the byte to be stored in the corresponding latch at the specified address in each matrix. The byte is now stored in memory. Later, if we want to read that byte, we input the address and activate the read enable signal. This causes the data lines to output the value of each bit that makes up the byte at that address.
![1750352451443](/assets/images/notes/computer_achitecture_part_2/1750352451443.png)
![1750352536705](/assets/images/notes/computer_achitecture_part_2/1750352536705.png)

When we write programs, we don't need to concern ourselves with the underlying electronic matrix etched into the hardware. Instead, we think of bytes as sequences of bits stored contiguously. Thus, we abstract this further and conceptualize memory as a list of bytes. In this abstraction, addresses aren't intersections in an unseen matrix, but rather indices in this list of bytes. This is called Random Access Memory, since we can point to any random byte in the list by providing its address.
![1750352630635](/assets/images/notes/computer_achitecture_part_2/1750352630635.png)
![1750353155001](/assets/images/notes/computer_achitecture_part_2/1750353155001.png)
![1750353180496](/assets/images/notes/computer_achitecture_part_2/1750353180496.png)

To simplify things, we can define these sets of wires as a bus. The width of the address bus determines how many bytes we can handle. For example, with an 8-bit address bus using two decoders, each capable of controlling 16 different outputs, we can locate any of 256 bits within the matrix.
![1750353245857](/assets/images/notes/computer_achitecture_part_2/1750353245857.png)

Generally, if the address bus has n inputs, 2 to the power of n bytes can be managed. For instance, a 32-bit wide address bus can address 4 billion bytes, which explains why 32-bit computers can only handle up to 4 GB of RAM.
![1750353335946](/assets/images/notes/computer_achitecture_part_2/1750353335946.png)

Finally, keep in mind that the type of memory we have learned about today is known as static RAM. This type of memory is very fast, but all these logic gates require quite a few transistors, so the production cost is higher. It turns out that those small memory cells in the matrix can also be achieved using a single special transistor called a MOSFET and a capacitor. It is cheaper to produce, but it is slower than static RAM. I'll make a video comparing these kinds of memory.
![1750353363740](/assets/images/notes/computer_achitecture_part_2/1750353363740.png)
![1750353382845](/assets/images/notes/computer_achitecture_part_2/1750353382845.png)

## Part 3 — The Control Unit: Fetch, Decode, Execute

Transistors are very simple by themselves, but combining them results in very powerful components like arithmetic logic units capable of doing math and latches capable of retaining information to create memory. In today's video, we are going to use these components to create our own CPU, so we'll finally understand how a piece of silicon runs programs.

Before starting, a quick reminder about registers. We discussed that a register is a set of tiny memory cells called latches made out of logic gates and interconnected in a way that retains information. Each latch in the register has its own output wire, which we use to read the value held in the latch. Registers also have input wires and a write enable wire to overwrite the values in the latches. When the write enable signal is off, the stored values do not change regardless of the input values. Activating the write enable signal is the only way to overwrite the values in the latches. Once set, the inputs can be deactivated, and the new values will remain stored.

Our first goal is to reduce the number of wires used. We can modify each latch so that the same wires serve as both inputs and outputs. By activating a read enable signal, these wires act as outputs allowing us to read the stored information. By activating the write enable signal, we use the same wires as inputs to overwrite the information in the register. This approach reduces the number of wires needed when working with multiple registers by combining their data lines into a single general data bus.
![1750491859961](/assets/images/notes/computer_achitecture_part_3/1750491859961.png)

When using this data bus as an input, the input value reaches each register. Since each register has its own write enable signal, we can activate the write enable signal of only the specific register we want to modify. The same principle applies to reading: activating the read enable signal for the particular register we want to read from causes the general data bus to act as an output. By selecting the correct write or read enable input, we can read from or write to any register using a single data bus.

Fortunately, we already know about the binary decoder, which can electronically select between multiple options.

Now, let's talk a bit about memory. In the previous episode, we defined memory as an array of bytes. Each byte can be selected by inputting its position in the array through an address bus. A data bus, alongside a read enable input, can be used to read the value stored in the selected byte. This same data bus can also be used as an input alongside a write enable input to overwrite the value of the byte at the provided address.

Our goal in this part of the video is to understand how instructions like load and store are executed at the hardware level. If you are not familiar with assembly language, don't worry—I’ll explain some of these instructions along the way.

Let's begin with load instructions. Typically, program data resides in memory; however, to manipulate this data, it often needs to be loaded into registers first. Within the CPU, there's a set of registers available for this purpose. The current instruction indicates that we want the computer to load into register one the value stored in memory location 1101. Essentially, a load instruction copies the value stored in a specific memory location into the designated register.

While this process might seem straightforward in animation, this video is about understanding the hardware implementation involving cables and logic components to make this possible. Let's get to it.

We'll start by connecting the memory data bus to each register. Suppose this is the instruction we need to execute. Since we aim to copy data from memory to a register, we start by reading the memory address where the data resides. This is accomplished using the address bus and activating the read enable signal of the RAM. Consequently, the memory data bus outputs the value stored at the provided address, making this value serve as input to every register.

Since we want this value to be copied only into register two, we activate the write enable signal only for that specific register, making it overwrite its current value with the data we are reading from memory.

On the other hand, with a store instruction, we perform the opposite action. For example, this instruction directs the computer to store the current value held in register one into memory location 0110. So now, instead of reading from a memory location, we need to read from a register—in this case, register one. Activating the respective read enable input causes the value stored in that register to become the input on the memory data bus.

Additionally, we must specify the memory address where we want to store the value. Since our objective is to write to memory, we activate the memory write enable signal. And just like that, we've successfully copied a value from a register to memory.

Quite straightforward, isn't it? Well, yes. But now we got the question: where do all the signals necessary to control these components come from? We already established that binary decoders can be used for this task, but then where do the inputs for those decoders come from?

The answer—after a quick message—is: where do the input signals for these components come from? The short answer is from the instructions themselves.

As discussed in a previous video, assembly language serves as a human-friendly representation of code that computers can understand. When we write assembly, an assembler—a specialized compiler—translates this human-readable code into binary machine code. This sequence of bits contains all the essential information to instruct the computer.

For example, in a previous video, we established that if the first two bits are 00, the computer interprets the instruction as an arithmetic operation. In that particular case, the third and fourth bits specify the type of operation. If the first two bits are not 00, the instruction signifies something other than an arithmetic operation. For example, it could be a load instruction if the first bits are 01, or a store instruction if they are 10.
![1750492209900](/assets/images/notes/computer_achitecture_part_3/1750492209900.png)

![1750492191844](/assets/images/notes/computer_achitecture_part_3/1750492191844.png)

![1750492233532](/assets/images/notes/computer_achitecture_part_3/1750492233532.png)

In this design, for load or store instructions, the third and fourth bits of the instruction don't indicate an operation code but rather which register to read from or write to. The last four bits represent the memory address.
![1750495047344](/assets/images/notes/computer_achitecture_part_3/1750495047344.png)

To distinguish between different kinds of instructions, we employ a binary decoder, and the third and fourth bits of the instruction serve as inputs for these other decoders so they can activate the enable signals of the desired register.
![1750495098030](/assets/images/notes/computer_achitecture_part_3/1750495098030.png)

However, in this configuration, both the read and write enable signals of the register are activated simultaneously, yet only one of them should be activated depending on the instruction. Since we've already designated a decoder to differentiate between different instruction types, we can use the outputs of that decoder to ensure that only the desired input is activated according to the instruction.
![1750495151451](/assets/images/notes/computer_achitecture_part_3/1750495151451.png)
This allows us not only to select a specific register but also to activate either its write or its read enable signal, but not both at the same time. Moreover, if these two bits are not meant to interact with any register, such as in the case of another type of instruction, this setup prevents any signal from reaching the registers, thus keeping their stored values safe.

Now, considering that the last four bits of the instruction represent the memory address we need to read from in the case of a load instruction, these bits will serve as input to the address bus. Finally, to copy the value of the selected address to the selected register, we simply activate the memory read enable signal. This action allows the data bus to output the value, sending it to the registers. Since only the write enable signal of register zero is active, only that register will overwrite its content with the received value.
![1750495397891](/assets/images/notes/computer_achitecture_part_3/1750495397891.png)

Let's do the same but now with the store instruction. As we already know, this instruction copies the value from a register to a specific memory location. Therefore, the decoder should activate only the read enable signal of the register specified in the third and fourth bit of the instruction—in this case, the read enable signal of register two. This action makes the value of that register act as an input on the memory data bus.
![1750495446307](/assets/images/notes/computer_achitecture_part_3/1750495446307.png)

It's worth noting that the same wire that activated the memory read enable signal in the previous load instruction is now deactivating it, which is precisely what we want. In a store instruction, we aim to write to memory instead. The last four bits of the instruction indicate where we want to write the value being input into the data bus, and to prompt memory to write that value into that location, the write enable signal of the memory should be activated.
![1750495505897](/assets/images/notes/computer_achitecture_part_3/1750495505897.png)

We achieve this by connecting the input to the corresponding output of the instruction decoder. And that's it—the value in register two is now copied to memory location 1111.
![1750495598677](/assets/images/notes/computer_achitecture_part_3/1750495598677.png)

Now, we already know that the input of all of these components comes from the instruction itself, but how exactly? Well, before executing an instruction, the instruction itself needs to be loaded into a special register called the instruction register. The output data lines of this special register are used as input for the components that decode the instruction. I'll get back to this register in a few moments.
![1750495792986](/assets/images/notes/computer_achitecture_part_3/1750495792986.png)

Before I do, I need to clarify something. Some of you might have noticed that this part of the circuit can be improved. Since both load and store instructions use the third and fourth bits to indicate the register to write to or read from, we could use a single decoder to achieve the same result.

There are two reasons I'm mentioning this. First, if I don't, someone in the comments will point out there's a better way to do it. Some have been arguing in the last few videos that modern hardware handles this in better ways. Once again, my response is: here, we're focusing on the fundamentals, not the fastest or most modern methods.
![1750495909363](/assets/images/notes/computer_achitecture_part_3/1750495909363.png)

Second, discussing these design decisions is when we start talking about architecture. Now, this term can be ambiguous, but let's focus on logical design for now. Different architectural designs can achieve the same result given the same instruction. From one perspective, we could say they are compatible architectures. But if we focus only on the instructions and their effects on the circuit, we could argue they are the same architecture.

This is not always the case. For example, consider two similar architectures where the only difference is the interconnections of the instruction register outputs. In the second one, this apparently trivial change causes both architectures to perform completely different actions.

While in the first architecture, the sequence of bits in the instruction register means store the current value of register two in memory location 15, in the second architecture, the same sequence of bits means load into register one the value stored in memory location 15. Thus, despite appearances, the architectures are incompatible.
![1750496019468](/assets/images/notes/computer_achitecture_part_3/1750496019468.png)

If you've ever wondered why a program compiled for Intel x86 machines cannot run on a Mac or Raspberry Pi with ARM chips—even though all are computers that understand ones and zeros—well, that's the reason. While the executable file is a list of instructions the Intel CPU can follow, that same file is a meaningless sequence of ones and zeros that other architectures just can't understand.
![1750496116628](/assets/images/notes/computer_achitecture_part_3/1750496116628.png)

Let's continue with the main topic we were talking about—the instruction register. But before understanding how it works, let's see how our architecture executes arithmetic operations. We'll omit unnecessary components for simplicity.

As described earlier, the first two bits being 00 indicate an arithmetic operation to perform. To do this, we'll use the arithmetic logic unit (ALU) we developed in the first episode of this series.

If the instruction is an arithmetic operation, the third and fourth bits specify the type of operation—in this case, subtraction. These two bits will serve as the opcode input for the ALU.

The ALU needs two values to operate, so we need to read from two registers simultaneously, accessing each register's data line directly. This requires internal logic to bypass the general data bus used previously.
![1750496232835](/assets/images/notes/computer_achitecture_part_3/1750496232835.png)

The fifth and sixth bits of the instruction determine the first operand, while the seventh and eighth bits determine the second operand. But we can't connect the registers directly to the ALU because its inputs could be any of the registers. We need circuitry that allows us to select which signals to send.

This is achieved by using a multiplexer, which internally employs a binary decoder to select which input bus value is sent to the output bus.

Now, to simplify the diagram, I'll assign a color to each register and draw its respective data bus in that color. We can use two multiplexers, one for each input of the ALU. The inputs of these multiplexers will be the individual data buses of each register.

The output of the instruction decoder, which identifies arithmetic operations, can toggle the read enable signal of all the registers, making them output their values through their respective data buses to serve as inputs to the multiplexers.

We then need to instruct each multiplexer on which register values to pass to the ALU—in this case, 2, 0, 5 and 5, 3. Since this is a subtraction operation, the ALU will output the difference between the input values, which in this particular example is 152.
![1750496390092](/assets/images/notes/computer_achitecture_part_3/1750496390092.png)

The result of the operation needs to be stored somewhere. In many architectures, it is stored in one of the registers that contained the operands. In our architecture, the result of any operation is stored in the register that contained the first operand.

But we can't write to it directly while reading from it. Instead, the result is temporarily saved in a register. Then, the registers are connected and set so the value can be written to the target register, which in this case is register one.

At this point, you've probably noticed that as we progress in this video, I'm starting to abstract more concepts and omit some details. This is intentional. I've done my best to provide a clear explanation. Unfortunately, I don't have enough screen space to draw every decoder, logic gate, multiplexer, or other component necessary to execute each possible instruction. Fifteen minutes wouldn't be enough either.

Hopefully, the overview of how the load and store instructions are decoded and executed gives you a brief idea of how this component, which we will call the control unit, works under the hood to decode a whole bunch of other instructions.

Now, let's explain how the control unit works. This is when we revisit the pending explanation about how instructions are loaded into the instruction register. This is perhaps the most important part of this video, so pay attention, because what I'm about to explain is practically how CPUs run programs.

The main job of the control unit is to read each instruction of a program and send the necessary signals to all other components to execute each particular instruction. In a way, the control unit acts like an orchestrator.
![1750496835283](/assets/images/notes/computer_achitecture_part_3/1750496835283.png)

Inside the control unit, there are special registers such as the instruction register and the address register. Additionally, there are small one-bit registers called flags, which will be very important in the next episode. All of these components are packed inside another component known as the central processing unit, also known as the CPU.
![1750496856152](/assets/images/notes/computer_achitecture_part_3/1750496856152.png)

When this giant circuit is powered on, everything is set to zero, so both data and instructions need to be fetched from an external source—memory. Unlike the general-purpose registers, the registers inside the control unit each have a specific purpose.

We already know the purpose of the instruction register. The address register's purpose is to point to the memory location of the next instruction the CPU will execute. Currently, that location is zero.
![1750497105084](/assets/images/notes/computer_achitecture_part_3/1750497105084.png)

To load the instruction into the instruction register, the control unit activates the memory read enable input so the memory can output the value stored at the provided address. In our architecture, this value is sent to multiple registers, so the control unit must activate the write enable input of the instruction register.

This process of bringing instructions from memory to the CPU is known as the fetch stage.
![1750497218326](/assets/images/notes/computer_achitecture_part_3/1750497218326.png)

Then comes the decode stage. The outputs of the instruction register are inputs for all the internal components of the control unit. In this step, the internal circuitry reads and interprets the bit sequence in the instruction register in order to send the necessary signals to all components needed to execute that specific instruction.
![1750497267125](/assets/images/notes/computer_achitecture_part_3/1750497267125.png)

And once everything is set up, the CPU can execute the instruction. In this example, copying the value from memory location 14 to register 0. This is known as the execute stage.
![1750497292947](/assets/images/notes/computer_achitecture_part_3/1750497292947.png)

And just like that, our CPU has run the first instruction of the program.

After executing the instruction, the internal circuitry of the control unit increments the value of the address register. In our architecture, the value is increased by one because each instruction is one byte long. Modern architectures have thousands of possible instructions, so instructions are much larger and the increment value can vary.
![1750497397958](/assets/images/notes/computer_achitecture_part_3/1750497397958.png)
This increment operation ensures that when the CPU goes back to the fetch step, the address register points to the next instruction of the program, and the cycle repeats.

The instruction is loaded into the instruction register, decoded, and then executed. Here, copying the value in memory location 15 into register 1. Afterward, the address register is incremented, pointing to the third instruction during the following fetch stage.

This time, the instruction is an arithmetic operation—specifically, an addition between the values in register 0 and 1, which were loaded in the previous two instructions. The control unit must activate the read enable signal of both registers and use internal multiplexers to direct both values to the ALU inputs.

The ALU also needs the opcode for the addition operation. The ALU then outputs the result of adding the two numbers.
![1750497531849](/assets/images/notes/computer_achitecture_part_3/1750497531849.png)
And, as we learned earlier, the result is temporarily stored in an internal register, giving the control unit time to reset and set the signals needed to store the value in the specified general-purpose register, which in this case is register zero.

After executing the instruction, the address register is incremented, and the cycle repeats.
![1750497557926](/assets/images/notes/computer_achitecture_part_3/1750497557926.png)
Since this is a store instruction, the control unit needs to set everything to copy the value from register zero, where the result of the addition is held, to memory location 13.

And there we go—we've come all the way from transistors to creating our own tiny CPU that runs programs.

If you think about it, the program we just ran is exactly what a line of code like this would look like when compiled for our tiny architecture. We load two values, add them, and then store the result in a variable. In this example, location 13 is where the content of variable A will reside in memory.

![1750497618168](/assets/images/notes/computer_achitecture_part_3/1750497618168.png)

Finally, for the people with more experience: I didn't mention the concept of a clock because it would make this video more complicated. I'll talk about clocks in a future episode.
![1750497633791](/assets/images/notes/computer_achitecture_part_3/1750497633791.png)

For now, I'll leave you with the question I mentioned at the beginning of the video.

## Part 4 — Dynamic RAM

About static RAM, a type of memory made entirely of transistors, today we are going to explore the basics of dynamic RAM—a different type of memory, cheaper to produce but with several challenges when implementing it.

Let's start talking about capacitors. A capacitor is an electronic device that stores electric charges. For example, if we connect its terminals to a voltage source, electricity will flow, and the capacitor will begin to accumulate energy, retaining it even after being disconnected from the power source. At this point, the capacitor acts like a small battery capable of powering another component, but only for a brief period—unlike a battery that can last for hours.
![1750581751206](/assets/images/notes/computer_achitecture_part_4/1750581751206.png)

![1750581760245](/assets/images/notes/computer_achitecture_part_4/1750581760245.png)

![1750581713765](/assets/images/notes/computer_achitecture_part_4/1750581713765.png)

![1750581723788](/assets/images/notes/computer_achitecture_part_4/1750581723788.png)
On the other hand, a MOSFET is a special kind of transistor, better suited for miniaturization and computer applications due to its efficiency compared to other types of transistors. It has three terminals: the gate, the source, and the drain. By applying a small voltage to the gate, electricity can flow from the source to the drain, and under certain conditions, electricity can also flow in the reverse direction, which will be important later on.

That being said, consider this: we can configure these two components in a way that uses the MOSFET's gate to allow electricity to flow, thus charging the capacitor. Once fully charged, we can interrupt the flow of electricity, and the capacitor will remain charged. Later, to discharge the capacitor, we can again use the gate to allow electricity to flow, but now in the opposite direction, enabling the charges to escape from the capacitor.
![1750581836168](/assets/images/notes/computer_achitecture_part_4/1750581836168.png)

![1750581856653](/assets/images/notes/computer_achitecture_part_4/1750581856653.png)

This very simple behavior is the fundamental of dynamic RAM. When the capacitor is discharged, we can consider a zero as being stored. When the capacitor is charged, a one is stored. This circuit can be abstracted as a memory cell capable of storing one bit of information.
![1750519164236](/assets/images/notes/computer_achitecture_part_4/1750519164236.png)

![1750519178065](/assets/images/notes/computer_achitecture_part_4/1750519178065.png)

![1750519195933](/assets/images/notes/computer_achitecture_part_4/1750519195933.png)
If you've watched my video on static RAM, you might think that dynamic RAM is way simpler to implement. In static RAM, we don't use capacitors; instead, we connect a series of logic gates in a clever but more complex arrangement to trap the value we want to store as an electrical signal. And when I say trap, I mean that the signal holds the value even if we reset the inputs to their default states.
![1750519287509](/assets/images/notes/computer_achitecture_part_4/1750519287509.png)

![1750581973132](/assets/images/notes/computer_achitecture_part_4/1750581973132.png)

Compared to this, charging and discharging a capacitor looks way easier, right? While a single static memory cell, also known as a latch, requires many transistors, we only need one transistor and a very small capacitor to create a dynamic memory cell. This is what makes dynamic RAM attractive: we can pack more memory cells into a given area compared to static RAM. In other words, we can achieve higher memory capacities using fewer components, making this type of memory very cost-effective to produce.

![1750582007256](/assets/images/notes/computer_achitecture_part_4/1750582007256.png)

![1750582043859](/assets/images/notes/computer_achitecture_part_4/1750582043859.png)

But not everything is perfect. Dynamic RAM has several disadvantages. Consider a scenario where a dynamic memory cell is storing a one, meaning the capacitor is fully charged. To read the value stored in the cell, our only option is to open the gate to check if electricity flows from the capacitor. This means that in dynamic memory, reading data is a self-destructive operation because checking whether a capacitor holds a charge involves discharging it.
![1750519396138](/assets/images/notes/computer_achitecture_part_4/1750519396138.png)

And this is where things start to become very complicated when implementing dynamic memory. For the rest of this video, we're going to explore how the reliability issues of dynamic memory are addressed.

A quick note for those more experienced viewers: I might simplify some concepts to keep the explanation accessible for everyone.

Firstly, it is impractical to handle each memory cell individually because that would require an insane amount of wires. Instead, we arrange the cells in a matrix. We use a single wire to connect the sources of each MOSFET along each column, which we'll refer to as bit lines, and a single wire connects the gates of the MOSFETs along each row, known as word lines.
![1750582120743](/assets/images/notes/computer_achitecture_part_4/1750582120743.png)

As mentioned in a previous episode, organizing memory cells in a matrix allows us to locate any cell by intersecting two wires. A component called a binary decoder, which utilizes a series of logic gates, enables us to select any given output. Using a decoder to select a row and another to select a column lets us pinpoint the specific intersection or memory cell we want to address. This introduces the concept of a memory address.

![1750582174201](/assets/images/notes/computer_achitecture_part_4/1750582174201.png)
However, using a decoder to select a bit line isn't ideal because bit lines not only carry data into the memory cells but also out of them during a reading operation. For example, activating a word line causes all cells in that row to output their values. In this scenario, we cannot send signals to the outputs of a decoder because, well, they are outputs.

![1750519609151](/assets/images/notes/computer_achitecture_part_4/1750519609151.png)

Moreover, remember that reading from a dynamic memory cell causes the capacitor to discharge. Since all cells in a row are connected to the same word line, any charged capacitors will discharge somehow, making the problem even worse.
![1750582319097](/assets/images/notes/computer_achitecture_part_4/1750582319097.png)

When things get this complicated, the solution can seem like magic—until someone with patience explains it in an intuitive way.

Dynamic RAM reads values from its cells without erasing its data in the process but uses 1 volt to represent a binary one and zero volts to represent a binary zero.
![1750582439014](/assets/images/notes/computer_achitecture_part_4/1750582439014.png)

One thing I haven't mentioned is that if the bit lines are disconnected from the ground and then a voltage is applied to them, they act as capacitors, storing charges and holding a voltage for a few nanoseconds.

The first step in reading is to pre-charge the bit lines to half the working voltage of the RAM—in this example, 0.5 volts.
![1750582508989](/assets/images/notes/computer_achitecture_part_4/1750582508989.png)

Why? When we use the decoder to activate a word line, we open the gate of all cells in that row. For cells holding a binary one, the capacitor's voltage is greater than the bit line's, so charges flow out, slightly discharging the capacitor. The bit line gaining some charges will see a small increase in voltage.
![1750582555783](/assets/images/notes/computer_achitecture_part_4/1750582555783.png)
For cells storing a binary zero, the completely discharged capacitor has a lower voltage than the bit line. As the gates are open, charges flow from the bit line to the capacitor, causing the bit line to lose some charges and its voltage to decrease.
![1750582603385](/assets/images/notes/computer_achitecture_part_4/1750582603385.png)
These voltage changes on each bit line are detected by a component called a **sense amplifier** at the end of each bit line. If the sense amplifier detects that the voltage is above 0.5 volts, it outputs 1 volt, representing a binary one. If below 0.5 volts, it outputs zero volts, representing a binary zero.

The internals of these sensors are beyond the scope of this video, but essentially they work by comparing two voltage sources. Each sensor is connected to a latch similar to those used in static RAM. Because the bit lines might not hold their charges long enough, as soon as a sense amplifier detects a value, it stores it in its latch.
![1750582672695](/assets/images/notes/computer_achitecture_part_4/1750582672695.png)

Even if the bit line discharges, the sense amplifiers can still output the values read from the specified row.

But we don't want the value of a whole row; we want the value of a specific cell in that row. Here, we use a component that gathers these outputs and uses the address input to determine which one is sent to the output line. This is exactly what a multiplexer does.
![1750582854522](/assets/images/notes/computer_achitecture_part_4/1750582854522.png)

Internally, a multiplexer, or mux, uses a decoder with a few logic gates connected in a clever way to achieve this behavior.
![1750582829110](/assets/images/notes/computer_achitecture_part_4/1750582829110.png)

After the sense amplifiers, a multiplexer is used to ensure we can read the value stored in any memory cell we want. But since reading alters the state of the capacitors storing those bits, we need to reset them to their original state.

This is straightforward: the sense amplifier reads the value from the latch and sends it back to the bit lines, restoring the capacitors to their previous state.
![1750582926104](/assets/images/notes/computer_achitecture_part_4/1750582926104.png)

And there you have it—we can read from dynamic memory without destroying our data in the process.

Let's now turn our attention to the writing operation, which builds on the basics we've already discussed.

We need to apply a voltage, either 0 or 1 volts, to the source of the MOSFET, then activate the word line to either charge or discharge the capacitor based on the desired stored value.
![1750583047670](/assets/images/notes/computer_achitecture_part_4/1750583047670.png)
![1750583063214](/assets/images/notes/computer_achitecture_part_4/1750583063214.png)

However, there are two challenges. First, we aim to write to a specific memory cell, not all the cells in the same row. Second, given the existing circuitry for the read operation, we can't simply bypass it.

Our solution is to modify this circuitry. Since we're writing, data should flow in the opposite direction; thus, a simple multiplexer won't work.

A demultiplexer, or demux, performs the opposite function of a multiplexer. It receives a single data input, and with a select input, it determines which one of the outputs will receive the input value.

We need a component that can act both as a multiplexer and a demultiplexer, commonly referred to as a mux-demux. This component's ability to switch roles depending on whether we are reading or writing is crucial for reliable memory operations.

The writing process closely resembles reading. We start by charging the bit lines to 0.5 volts, then activate the word line of the desired cell, causing all capacitors in that row to either charge or discharge and change the voltage of the bit lines. The sense amplifiers detect these changes and store the values just as they do during reading.

![1750583354284](/assets/images/notes/computer_achitecture_part_4/1750583354284.png)
![1750583383296](/assets/images/notes/computer_achitecture_part_4/1750583383296.png)

The key difference in writing is that instead of outputting a value, we input it. The address tells the demultiplexer which latch to direct the input value to. With only the targeted latch overwritten, the sense amplifiers then send all values back through the bit lines.
![1750583754463](/assets/images/notes/computer_achitecture_part_4/1750583754463.png)

This step ensures that each cell's capacitor either charges or discharges according to the value received. In other words, we write the new value while also rewriting other values in the same row to prevent data loss.

And this is how we write to any memory cell in the matrix without affecting the data integrity of other cells in the same row.

Here's another example where we're writing a binary zero to a different memory location. The process starts by charging the bit lines, then selecting a row. The sense amplifiers detect and store the existing values. The intended zero is then input and directed by the demultiplexer to the specific latch.

Once the value is overwritten, all values, including the newly changed one, are written back to the memory cells from which they originated.

Now, before we move on to the next issue with dynamic RAM—yes, we're not done yet—let's outline the steps for each operation.
![1750583852111](/assets/images/notes/computer_achitecture_part_4/1750583852111.png)

Note that except for the intermediate steps, both operations appear almost identical, although all of this occurs in just a few nanoseconds.

There are many steps involved in reading or writing a single bit of information, especially when compared to static RAM.

In static RAM, bits are stored as electrical signals rather than charges, so reading from a latch doesn't cause it to lose its value. This is a significant reason why dynamic RAM is slower than static RAM.

Despite the process taking only nanoseconds, these extra steps slow down the entire memory addressing process.

If you want to learn more about how static RAM handles this, check out my previous video.

Finally, there is one more problem with dynamic RAM that I haven't mentioned.

Even when the gate of the MOSFET is closed, some small charges can still escape, moving from the drain to the source of the MOSFET, thus discharging the capacitor.
![1750583949148](/assets/images/notes/computer_achitecture_part_4/1750583949148.png)

Although this happens much slower than if we intentionally open the gate of the MOSFET, it's still a major issue.

If we don't address any cell in a row, the information contained by those cells will be lost over time as we are neither reading nor writing to refresh the information at the end of the operation.

The solution to the problem is to make this type of memory refresh itself.

From this point, I will abstract some things to avoid overcomplication.

With everything we've learned so far, understanding this operation called refreshing should be easy.

Before the capacitors in a row have time to discharge to the point where we can no longer read their information, we simply read all of the values in that row and then set the bit lines to the voltage that corresponds to those values.

This ensures the cells are fully charged or discharged according to the binary values they were originally storing.

Essentially, we are doing the same as we did for the reading and writing operations but without outputting or inputting any values—so no multiplexer or demultiplexer is used during this process.

In a few words, refreshing works by reading the value in a row and then writing those values back to the cells they came from.

Since the circuit has only one latch per bit line, not all cells in the matrix can be refreshed simultaneously. Thus, each row is refreshed one by one.
![1750584193873](/assets/images/notes/computer_achitecture_part_4/1750584193873.png)
![1750584200551](/assets/images/notes/computer_achitecture_part_4/1750584200551.png)

![1750584219520](/assets/images/notes/computer_achitecture_part_4/1750584219520.png)
![1750584227040](/assets/images/notes/computer_achitecture_part_4/1750584227040.png)

![1750584252885](/assets/images/notes/computer_achitecture_part_4/1750584252885.png)
![1750584261757](/assets/images/notes/computer_achitecture_part_4/1750584261757.png)
Special circuitry is designed specifically for this purpose.

Ideally, reading and writing operations are performed when none of the rows are being refreshed. But this introduces another disadvantage.

In dynamic RAM, each refresh cycle temporarily prevents access to the memory since the same circuitry needed for reading or writing is used for the refresh operation.

If the CPU attempts to read or write during a refresh cycle, it must wait until the cycle is complete, introducing latency.

The good news is that typically, the intervals during which no row is being refreshed are long enough to allow thousands of reading and writing operations before the next refresh cycle begins.

This is because each row generally needs to be refreshed every few milliseconds, while reading and writing operations usually take nanoseconds.

![1750584408994](/assets/images/notes/computer_achitecture_part_4/1750584408994.png)
![1750584414997](/assets/images/notes/computer_achitecture_part_4/1750584414997.png)

I'm a software engineer, not an electrical engineer, so I hesitate to provide exact figures.

For more details on aspects I don't feel fully qualified to explain, such as the timing involved in dynamic RAM operation or specific details about DDR5 RAM, I highly recommend checking out these videos.
![1750584485315](/assets/images/notes/computer_achitecture_part_4/1750584485315.png)

Before getting into the final part of this video, I want you to think about one of the examples on reading or writing operations.

You might have noticed before that when we input an address, different parts of the address are not used at the same time.

For example, in the writing operation, the part of the address that goes to the decoder is clearly used before the part that goes to the multiplexer.

When the CPU sends an address to read or write to, it cannot be connected directly. Rather, it is intercepted by another component that handles all of these timing matters, knowing exactly when to let each signal reach its target component depending on the operation the CPU wants to perform.

My point here is that there is a lot of timing involved in these operations, and there is some complicated circuitry that I'm not showing.
![1750584652640](/assets/images/notes/computer_achitecture_part_4/1750584652640.png)

Finally, some of you might have noticed that up until now, we've been working with individual bits. However, computers use bytes, so how do we address bytes instead of bits?

This is quite simple.

Part of the answer lies in the complexity of multiplexers. Rather than routing individual bits of information, they can handle entire data buses internally.

This is still just a binary decoder with a few logic gates.

With a more complex multiplexer, we can add more columns to the matrix and divide them into groups of eight, since one byte equals eight bits.

In a reading operation, for example, the address tells the multiplexer which group of values to send to the output.
![1750584709714](/assets/images/notes/computer_achitecture_part_4/1750584709714.png)

For writing operations, a demultiplexer capable of managing buses instead of single data lines is needed.

In conclusion, dynamic RAM has several disadvantages over static RAM, mainly because capacitors are used to store information and managing these capacitors to avoid losing information involves numerous steps that require precise timing.

But even though it's slower than static RAM, dynamic RAM is cheaper to produce because it allows higher storage capacities per area.

Hopefully, this video has clarified that the cache inside your processor—a type of static RAM—is faster than your computer's dynamic RAM memory modules due not only to their proximity to the CPU but also to their fundamentally different operational mechanisms.
![1750584913787](/assets/images/notes/computer_achitecture_part_4/1750584913787.png)
So, the next time you hear the term locality, don't take it too literally. Connecting your hard drive closer to the CPU won't make it faster than your RAM.

