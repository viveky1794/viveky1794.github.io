**Understanding Transistors and Logic Gates**

Transistors are very simple components; they are basically electronic switches. When we apply current to one of its terminals, a transistor lets electricity pass through. But how can this simple behavior make computers possible? In today's video, we are going to learn how transistors can be used to achieve more complex tasks like doing math and even interpreting instructions.

A single transistor typically has three terminals: a collector, an emitter, and a base. In a circuit, it can act as an insulator, preventing electricity from flowing between the collector and emitter terminals.
![1750172983097](image/computer_achitecture_part_1/1750172983097.png)

But if we apply a small current to the base terminal, it acts as a conductor, allowing electricity to flow. Essentially, a transistor can be seen as a switch, but instead of mechanical movement, it operates by using electrical signals.
![1750174244574](image/computer_achitecture_part_1/1750174244574.png)

In this example circuit, we are using a transistor to turn on and off an LED. By utilizing the base terminal as an input and the emitter terminal as an output signal, we can mimic the input signal with a switch and visually represent the output signal with the LED.
![1750174425671](image/computer_achitecture_part_1/1750174425671.png)
![1750174458663](image/computer_achitecture_part_1/1750174458663.png)
![1750174477806](image/computer_achitecture_part_1/1750174477806.png)

Let's dub this a simple gate. In a simple gate, when the input is zero, the output is zero. When the input is one, the output is one.
![1750174498793](image/computer_achitecture_part_1/1750174498793.png)
![1750174508000](image/computer_achitecture_part_1/1750174508000.png)

Now let's tweak things a bit here. When the input is zero, the LED is on due to the way it is arranged within the circuit. But pay attention to this: setting the input to one causes the LED to turn off, indicating to us that the output is a zero. This setup is commonly referred to as an inverter or a NOT gate.
![1750174553266](image/computer_achitecture_part_1/1750174553266.png)
![1750174577068](image/computer_achitecture_part_1/1750174577068.png)

This may seem a bit confusing. This video I found on Twitter is a perfect example of this effect, and if you want more details, you can watch this video by Ben Eer, where he explains all this using real components.
![1750174648547](image/computer_achitecture_part_1/1750174648547.png)
![1750174676590](image/computer_achitecture_part_1/1750174676590.png)

We're not limited to just one transistor. By using multiple transistors, we can achieve more complex behavior. If two transistors are connected in series, when both inputs are zero, the output will be zero because both transistors act as insulators.
Similarly, if either input is zero, the output will be zero. The only way to obtain a one in the output is by having both transistors act as conductors, which happens when both inputs are set to one.
![1750174740391](image/computer_achitecture_part_1/1750174740391.png)
![1750174747081](image/computer_achitecture_part_1/1750174747081.png)
![1750174754992](image/computer_achitecture_part_1/1750174754992.png)
![1750174766677](image/computer_achitecture_part_1/1750174766677.png)

This is where we begin to apply a powerful concept called abstraction. Instead of focusing on the individual transistors in the circuit, we can abstract this into a white box—a box that consistently outputs a specific value based on two given inputs. This is known as an AND gate. An AND gate outputs a value of one if and only if both inputs are one; otherwise, the output will be zero.
![1750174830021](image/computer_achitecture_part_1/1750174830021.png)
![1750174842491](image/computer_achitecture_part_1/1750174842491.png)

If we connect the transistors in parallel, when both transistors act as insulators, electricity cannot flow. But in this setup, having any transistor acting as a conductor is enough to allow electricity to flow. So in this arrangement, to get an output of one, it is not strictly necessary to set both inputs to one. Once more, we can abstract this circuit into a box known as an OR gate. An OR gate outputs a value of zero if and only if both inputs are zero; otherwise, it outputs a value of one.
![1750174920397](image/computer_achitecture_part_1/1750174920397.png)
![1750174947603](image/computer_achitecture_part_1/1750174947603.png)

These circuits, known as logic gates, are so fundamental that instead of representing them with boxes, each one is assigned a dedicated symbol. Notice that since inputs and outputs are electrical signals, we can connect the output of one logic gate to the input of another. This allows us to combine logic gates to achieve even more complex behavior. For example, if we desire this particular behavior, we can combine logic gates accordingly to achieve it. This is where we begin to see the power of abstractions. While this setup is ultimately composed of transistors, it's much simpler to understand what's happening by thinking in terms of logic gates.
![1750175013262](image/computer_achitecture_part_1/1750175013262.png)
![1750175022369](image/computer_achitecture_part_1/1750175022369.png)

This circuit is known as an XOR gate, and it is also very important in computer science, so it has its own symbol.
![1750175128974](image/computer_achitecture_part_1/1750175128974.png)
![1750175147212](image/computer_achitecture_part_1/1750175147212.png)
![1750175167328](image/computer_achitecture_part_1/1750175167328.png)

 And now that we understand what logic gates are, let's use them to create more useful things. Let's start with adders. Adding binary numbers is actually quite simple. In fact, binary addition works just like decimal addition: 0 + 0 = 0, 0 + 1 = 1, 1 + 0 = 1, and 1 + 1 = 2. However, the value two cannot be represented with a single binary digit. When this occurs, we say the addition has overflowed, meaning we require an additional bit to represent the value.
![1750175257766](image/computer_achitecture_part_1/1750175257766.png)

So what we're aiming for is a circuit that takes two input values and produces two outputs: the sum and the carry. Let's break down the sum. But before, a quick message from [Your Sponsor]. We require a circuit that outputs zero when both inputs are the same and one when they differ. Does this sound familiar? It's precisely what an XOR gate accomplishes, so we've already solved half of the puzzle.
![1750175412504](image/computer_achitecture_part_1/1750175412504.png)
![1750175427012](image/computer_achitecture_part_1/1750175427012.png)
![1750175447898](image/computer_achitecture_part_1/1750175447898.png)

For the carry output, we need a circuit that outputs the value one only when both inputs are one. This matches the behavior of an AND gate, and there we have it! We've crafted a circuit capable of adding two single-digit binary numbers.
![1750175495520](image/computer_achitecture_part_1/1750175495520.png)

Okay, but what about multi-digit binary addition? Well, let's try. Since we have to add numbers in each row, using one adder for each row seems logical. It works well for the first row, but when we move to the second row, there's an issue. We need to consider that the previous row might have created a carry, but because the adder circuit only accepts two inputs, it doesn't account for this carry. Because of this limitation, this circuit is known as a half adder, and it is not useful in this scenario.
![1750175671844](image/computer_achitecture_part_1/1750175671844.png)
![1750175691976](image/computer_achitecture_part_1/1750175691976.png)

What we need is a full adder—a circuit that can handle not only the two bits being added but also take into account the carry from a previous addition. Here's the circuit we're working towards. Think of it as a circuit capable of adding three single-digit binary numbers.
![1750175810815](image/computer_achitecture_part_1/1750175810815.png)
![1750175857317](image/computer_achitecture_part_1/1750175857317.png)
![1750175872312](image/computer_achitecture_part_1/1750175872312.png)

To simplify matters, we can encapsulate this in a box, which we'll call a full adder. With full adders, we can now add multi-digit binary numbers. The carry output of each full adder feeds directly into the carry input of the next full adder, as shown here in this animation.
![1750175923572](image/computer_achitecture_part_1/1750175923572.png)

To add binary numbers of n digits in one go, n full adders are needed. For example, if we aim to add two 8-bit numbers, we'll need eight full adders.
![1750175952509](image/computer_achitecture_part_1/1750175952509.png)

![1750176494031](image/computer_achitecture_part_1/1750176494031.png)
**Remember that values are represented here using electrical signals. Since electricity moves at incredible speed, once we alter the input, the output changes almost instantly, and this is what makes transistors ideal for this job. While logic gates can be crafted from other components like relays and even fancy 3D printed parts powered by weird stuff like marbles or even water, none of this matches the speed and compactness of transistors.**

Before things become overly complex, we can one more time package all of this functionality into a special component known as an 8-bit adder. An 8-bit adder takes two 8-bit numbers as input and produces the sum of the inputs as another 8-bit number. It also provides an overflow signal, which is essentially the carry-out output of the last full adder. This overflow signal is crucial because it informs us whether the storage capacity being utilized is adequate to represent the result of the operation. In this example, both inputs are one byte long, but if we attempt to store the output in just one byte, we will miss information, resulting in an incorrect value. By monitoring the overflow output, we can recognize the need for an extra byte to accurately store the output.
![1750176588132](image/computer_achitecture_part_1/1750176588132.png)
![1750176607623](image/computer_achitecture_part_1/1750176607623.png)
![1750176620284](image/computer_achitecture_part_1/1750176620284.png)
![1750176634081](image/computer_achitecture_part_1/1750176634081.png)

Believe it or not, neglecting to manage operation overflows can lead to undefined behavior, sometimes with severe consequences, like this rocket accident in 1996. And we can continue to move to higher levels of abstraction. If we want to make a circuit that increments an input value, we can simply use an adder and set the second input to always be one. We've only talked about adders in detail, but in the end, it's all about logic gates. With them, we can make more useful things like a full subtractor that then can be used to build an 8-bit subtractor, and from there, we could keep going and make even more complicated stuff.

As you may have guessed, inside the CPU, there's a special component that houses all these circuits. For now, let's call it our mysterious component. The question we want to answer now is: when a CPU reads an instruction, how does it identify which one of these circuits corresponds to that specific instruction? How does the computer discern adding numbers upon encountering a particular instruction and subtracting them upon encountering another? I mean, some instructions aren't even arithmetic operations.
![1750176751695](image/computer_achitecture_part_1/1750176751695.png)
![1750177883459](image/computer_achitecture_part_1/1750177883459.png)
![1750177894180](image/computer_achitecture_part_1/1750177894180.png)

**The last type of component we are covering today are binary decoders.** **Let's take a look at this circuit and examine the output for every input combination. The first thing we can notice here is that each combination triggers a specific output to activate while deactivating all other outputs. Another way to see it is that the circuit receives the binary number that corresponds to the position of the output we wish to activate. This is what binary decoders do:** when it receives an input, one and only one output has the value of one, with all others outputting the value of zero. If we have a decoder with three inputs, we can control which of eight outputs turns on; with four inputs, we can control 16 outputs, and so on.
![1750177936001](image/computer_achitecture_part_1/1750177936001.png)
![1750177976774](image/computer_achitecture_part_1/1750177976774.png)
![1750178004733](image/computer_achitecture_part_1/1750178004733.png)
![1750178016707](image/computer_achitecture_part_1/1750178016707.png)

This is huge because it means that we are also capable of creating circuits that can select among multiple options. **Now remember that assembly code is just a human-friendly representation of machine code—the actual code consisting of ones and zeros that computers comprehend. Not all instructions are arithmetic operations; some of them are instructions dedicated to fetching and writing data to memory and other stuff like that.**
![1750178058554](image/computer_achitecture_part_1/1750178058554.png)

***Let's imagine a very basic architecture where if the first two bits of an instruction are zeros, the computer interprets them as arithmetic operation instructions***.
![1750178088035](image/computer_achitecture_part_1/1750178088035.png)

In this example, to verify this, we could employ NOR gates. Given the scope of this video, at the moment we are not concerned with instructions that signify something else, but at least we know how to identify between them. In this example architecture, the third and fourth bits will determine the type of arithmetic operation to be executed. **For instance, if those bits are 00, it means addition; if 01, it represents subtraction, and so forth. This is commonly known as an opcode. Each opcode is associated with one and only one kind of arithmetic operation.**
![1750178167826](image/computer_achitecture_part_1/1750178167826.png)
![1750178336364](image/computer_achitecture_part_1/1750178336364.png)

**When the CPU has determined that the current instruction is an arithmetic operation, our mysterious component receives this opcode and internally links those two bits to a decoder, which is used to identify the desired internal operation. The straightforward approach here would be by allowing all circuits to receive the inputs and generate their respective outputs, but the outputs of the decoder are interconnected in a way that allows only the output of the selected operation to pass through this component. Keep in mind that there are more efficient ways to do this, but here we focus on simplicity.**
![1750178373459](image/computer_achitecture_part_1/1750178373459.png)
![1750178416710](image/computer_achitecture_part_1/1750178416710.png)
![1750178444979](image/computer_achitecture_part_1/1750178444979.png)
![1750178469357](image/computer_achitecture_part_1/1750178469357.png)
**Our mysterious component can also be enclosed within a box. This is a rudimentary and somewhat incomplete version of something known as an arithmetic logic unit.** We'll talk about this component in more detail in a future episode where we discuss how CPUs execute instructions. But beforehand, **an arithmetic logic unit takes input values and an opcode that tells the internal circuitry what arithmetic operation to perform between those values. It then produces the result of the specified operation along with additional information such as whether the result is negative, zero, or if it has overflowed.**
![1750178484219](image/computer_achitecture_part_1/1750178484219.png)
