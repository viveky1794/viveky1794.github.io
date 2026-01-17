learned how transistors can be used to create more complex components such as adders, decoders, and even a basic arithmetic logic unit, but we didn't cover how computers remember information. So, hold on to your seat, because in this video we are going to create memory from transistors.

Hi friends, my name is George. Transistors are used to create logic gates and then combine logic gates to achieve more complex behaviors, with inputs and outputs typically flowing in the same direction—meaning signals move forward. But when discussing memory, we're referring to the ability to retain information, so we might instinctively think about what occurs when we take an output and connect it back to the input of the circuit.
![1750348679812](image/computer_achitecture_part_2/1750348679812.png)
![1750348778995](image/computer_achitecture_part_2/1750348778995.png)

Let's examine the behavior of an OR gate in this situation. We establish a default state where everything is off. Remember, the output of an OR gate is zero if and only if both inputs are zero. When we set the first input of the gate to one, the output changes accordingly, and for a brief moment measured in nanoseconds, the circuit looks like this. 
![1750349054266](image/computer_achitecture_part_2/1750349054266.png)

But because electricity moves extremely quickly, this output value propagates back to the input almost instantly, changing the second input to one. Now, with the second input of the OR gate set to one, the output remains one regardless of the value we set for the first input, as long as the transistors are powered, of course. So, in other words, this circuit is remembering that the output was set to one at some point.
![1750349092247](image/computer_achitecture_part_2/1750349092247.png)

![1750349132624](image/computer_achitecture_part_2/1750349132624.png)

With the AND gate, we can get something similar. Recall that an AND gate outputs one only when both inputs are one.
![1750349171995](image/computer_achitecture_part_2/1750349171995.png)

If we start from this state and then change the first input to zero, the output will become zero.
![1750349217544](image/computer_achitecture_part_2/1750349217544.png)
Again, for a brief duration, the circuit resembles this configuration, but the output immediately propagates back to the second input. Now, with one of its inputs permanently set to zero, the gate will consistently output zero regardless of the value we set in the other input. Thus, in a sense, we could say the circuit remembers that its output value was set to zero.
![1750349239253](image/computer_achitecture_part_2/1750349239253.png)

![1750349251558](image/computer_achitecture_part_2/1750349251558.png)

Now we have two circuits: one capable of remembering a zero and the other capable of remembering a one. However, individually, they are not particularly useful, because what we need is a circuit that can remember either a one or a zero, not just one of them. Intuitively, our next step is to attempt combining both circuits and observe the outcome.
![1750349329680](image/computer_achitecture_part_2/1750349329680.png)

![1750349338688](image/computer_achitecture_part_2/1750349338688.png)

By default, the circuit looks like this. The value held by the circuit depends ultimately on this AND gate. In order to get an output of one, we can set both inputs to one, and since the output goes back to the OR gate, we can reset the first input back to zero without changing the output—but not the second input, because it goes directly to the AND gate.
![1750349389515](image/computer_achitecture_part_2/1750349389515.png)

![1750349408772](image/computer_achitecture_part_2/1750349408772.png)

![1750349467286](image/computer_achitecture_part_2/1750349467286.png)

Let's add an inverter to the second input and see what happens.
![1750349553850](image/computer_achitecture_part_2/1750349553850.png)

![1750349611776](image/computer_achitecture_part_2/1750349611776.png)

By default, the circuit looks like this. Now, with this arrangement, if we set the first input to one, the output of the AND gate will indeed be one, because this inverter ensures that the other input of the AND gate is one even though the second input of the circuit is zero. The output of the AND gate, directly connected to one of the inputs of the OR gate, ensures that setting the first input back to zero will still retain the value of one in the output.
![1750349663257](image/computer_achitecture_part_2/1750349663257.png)

![1750349704988](image/computer_achitecture_part_2/1750349704988.png)

Similarly, if we set the second input to one while keeping the first input at zero, the inverter will cause the AND gate to output zero, and once again, the output feeding back to the input of the OR gate will make the circuit maintain a value of zero. Because even if we set the second input back to zero, the AND gate needs its two inputs to be one in order to output one, so it will retain a zero in the output instead.
![1750349859290](image/computer_achitecture_part_2/1750349859290.png)

In other words, this circuit can remember one bit of information. Whenever we need to determine the current value held in the circuit, all we need to do is read the value from this wire. And there we have it—we've successfully created memory. We can encapsulate this circuit within a box that we'll refer to as an AND-OR latch. The first input allows us to set the stored value to one, while the second input allows us to reset the stored value back to zero.
![1750349946664](image/computer_achitecture_part_2/1750349946664.png)

![1750349980693](image/computer_achitecture_part_2/1750349980693.png)

![1750350064136](image/computer_achitecture_part_2/1750350064136.png)

![1750350074316](image/computer_achitecture_part_2/1750350074316.png)

![1750350090214](image/computer_achitecture_part_2/1750350090214.png)


Unfortunately, with this setup, things get a little weird when we set both inputs to one at the same time. I encourage you to explore what happens when you attempt to do so. Short answer: it doesn't make sense to try to set the stored bit to one while simultaneously resetting it back to zero. If we desire a more intuitive mechanism, we require additional circuitry that enables us to specify which value we want to store using a single input. After all, a single wire can transmit either a one or a zero, right?
![1750350121525](image/computer_achitecture_part_2/1750350121525.png)

Well, yes, but there's a complication. As soon as the input changes, the output value changes as well, so it no longer fulfills the function of memory. Instead, it simply outputs the current input value.
![1750350184166](image/computer_achitecture_part_2/1750350184166.png)

![1750350193552](image/computer_achitecture_part_2/1750350193552.png)

To address this issue, we introduce another input that we can activate whenever we wish to set the value being remembered. We'll name this new input as WR enable. With the addition of a few extra logic gates, we can achieve a more useful behavior.
![1750350255046](image/computer_achitecture_part_2/1750350255046.png)

![1750350268915](image/computer_achitecture_part_2/1750350268915.png)

When the write enable input is set to zero, the value in the data signal is completely ignored and nothing will happen. However, when the write enable input is set to one, the circuit will retain whatever value is being passed to the data input—in this example, one—and the value will persist even if we set the inputs back to zero later. If there comes a time when we need to overwrite the value stored in the circuit, all we have to do is activate the write enable signal while inputting the desired new value we wish to store—in this case, zero. As long as the transistors that make up this gate are connected to the current, the value will be remembered.
![1750350348677](image/computer_achitecture_part_2/1750350348677.png)

![1750350375035](image/computer_achitecture_part_2/1750350375035.png)

![1750350388563](image/computer_achitecture_part_2/1750350388563.png)

![1750350407740](image/computer_achitecture_part_2/1750350407740.png)

![1750350425517](image/computer_achitecture_part_2/1750350425517.png)

One more time, we can abstract this circuit into a component known as a gated latch. And before continuing, I just want to clarify that there are several other ways to create latches; what I'm showing you here is not the only way to create circuits capable of remembering information. A gated latch is capable of storing one bit of information. If its write enable input is set to zero, no action will occur, but if the WR enable input is set to one, it will retain whatever bit of information is passed through the data input.
![1750350542844](image/computer_achitecture_part_2/1750350542844.png)

With just a single bit of information, we can represent basic data like one or zero, true or false, or simple distinctions like black or white—but that's it, nothing too sophisticated. If we aim to represent a broader range of numbers or a larger set of colors, a single bit becomes insufficient—well, not really useful. But we can use multiple latches to expand the range of values we can store. For example, to store one byte of information, we can employ eight latches and combine all of their write enable inputs into a single input that we can use whenever we want to override all of the individual latches at the same time. A collection of latches interconnected in this way is known as a register—in this example, an 8-bit register.
![1750350816409](image/computer_achitecture_part_2/1750350816409.png)
![1750350844125](image/computer_achitecture_part_2/1750350844125.png)
![1750350856017](image/computer_achitecture_part_2/1750350856017.png)

If you've seen other videos on my channel, you've likely come across the concept of registers. In previous episodes, I've discussed various types, such as general-purpose registers, the instruction register, the address register, and the stack pointer. Registers are often likened to variables integrated directly into the hardware circuitry, which is a solid analogy, although somewhat abstract. Ultimately, they all consist of logic gates composed of transistors interconnected in a clever manner to retain values.
![1750350895610](image/computer_achitecture_part_2/1750350895610.png)
![1750350902595](image/computer_achitecture_part_2/1750350902595.png)
![1750350923418](image/computer_achitecture_part_2/1750350923418.png)

The primary challenge with registers is their demand for numerous wires. The quantity of latches within a register determines its length, which can vary. Registers come in different sizes. However, larger registers imply an increase in both inputs and outputs. For instance, achieving a 1 GB storage capacity would require a big register with millions of wires for data inputs and outputs. If we want to remember bigger amounts of data, we need another solution.

Instead of arranging latches in a linear array, we'll organize them into a 4x4 matrix—totaling 16 latches. Our task now is to establish a method for identifying each one. To accomplish this, we'll utilize four wires for columns and four for rows. By activating a single wire in both the column and row, we can pinpoint a unique latch in the matrix. If we think about it, this mirrors the process of locating addresses on a map, where streets and avenues are typically numbered, so we can say each latch in the matrix has an address.
![1750351042831](image/computer_achitecture_part_2/1750351042831.png)

Next, we need special circuitry to activate these wires in the described way. Fortunately, we've previously explored decoders. A decoder, when given an input, activates only one of its outputs. For instance, inputting 0 activates output zero; inputting 01 activates output one, and so forth. By employing one decoder for rows and one for columns, we can input an address to select a specific latch in the matrix. For example, inputting 000000 activates the first output of each decoder; inputting 00001 activates the first output of the column decoder and the second output of the row decoder, thereby selecting a different latch in the matrix.
![1750351103155](image/computer_achitecture_part_2/1750351103155.png)
![1750351119272](image/computer_achitecture_part_2/1750351119272.png)

Each latch possesses its unique address, which gives rise to the term memory address. Now that we can identify each latch in the matrix by its address, our next step is to devise a method for both reading and writing its value using the fewest wires possible. We can start by adding a few extra components to each latch so we can use a single wire both as input and output. To do this, we create a new input to let us toggle between reading and writing to the latch. Activating the read enable signal configures the data line as an output.
![1750351233297](image/computer_achitecture_part_2/1750351233297.png)
![1750351316833](image/computer_achitecture_part_2/1750351316833.png)


Now, someone might ask in the comments about the output signal feeding back into the input of the latch during reading. This is not a problem, since the write enable signal is inactive, thus disregarding any input. When the read enable input is off, the transistor prevents the signal from reaching the output, allowing the data line to serve as an input. With the data line functioning as an input, we can set the write enable data to one, overwriting the current value stored in the latch. All latches within the matrix can be modified in this manner.
![1750351448958](image/computer_achitecture_part_2/1750351448958.png)


Having the required number of data wires—instead of 32 wires (16 inputs and 16 outputs), we only need 16 wires that can be toggled using the write enable and read enable inputs. But we can take this further by connecting the data lines of all latches to a single general data line, the write enable input of each latch to a single write enable line, and the read enable input of all latches to a single read enable line. We can set the value of all latches in the matrix without employing 16 data input wires and 16 write enable wires. Making all the latches remember the same value is not the behavior we want, though—we need each latch to remember its own value.
![1750351619599](image/computer_achitecture_part_2/1750351619599.png)

![1750351646496](image/computer_achitecture_part_2/1750351646496.png)

![1750351693383](image/computer_achitecture_part_2/1750351693383.png)
Fortunately, we are almost there, because we already know how to use decoders to activate the cables intersecting precisely where each latch resides. We can use the trusty old AND gate to make some clever modifications to ensure that the write enable and read enable signals only reach the intended latch specified in the address input. As soon as the address input changes, the AND gate obstructs any passage of input signals through this point. So, in this example, even though the data input of the latch is set to zero, the latch will retain the value one in its memory because its write enable input is zero.

When all of the latches are connected in the same way, we get this—beautiful, isn't it? Let's see this circuit in action. Setting the write enable input to one allows the signal to traverse the matrix, yet it won't reach any specific latch due to the AND gates blocking its path. For the signal to target a particular latch, an address must be provided, prompting the decoders to activate one wire each for the columns and rows. The sole AND gate producing an output of one, thereby enabling the signal to flow to the latch, is where the active outputs of the decoders intersect. That's why this latch just changed its value from one to zero, as dictated by the value on the data line now utilized as input.

To revert the value of this latch back to one, we simply maintain this configuration while inputting one into the data line. If the address input changes, the output of the decoders will change, causing the activated outputs to intercept at a different location, and the write enable signal will exclusively target that specific latch, thereby overwriting its value with the data passed through the data line.

If we intend to read the value of a specific latch, we supply an address and activate the read enable signal. This will make the signal traverse the matrix but exclusively target the latch specified by the address, making the data line act as an output. And this is how we read the value any specific latch in the matrix is currently remembering. 
![1750351798051](image/computer_achitecture_part_2/1750351798051.png)
It's noteworthy that the value of the latch being read is propagated throughout the matrix, directly reaching the input of all other latches. Since none of the latches in the circuit have their write enable input activated, this value remains disregarded, so the value contained in the other latches won't be overwritten.

Okay, what you see here is actually an oversimplification, and even so, this is starting to get too complicated. I might have gone too far this time, so let's abstract all of this. With the address input, we can select any of the internal latches; by using the read enable signal, we make the data line output the current value of the selected latch. Conversely, if we use the write enable signal, we can override the value in the selected latch using the data line as input.
![1750352336455](image/computer_achitecture_part_2/1750352336455.png)

Now, we are still dealing with single bits. You probably already know that the minimum data computers handle are bytes. To store bytes instead of bits, we can use eight matrices. All these matrices will share the same address input, allowing us to select corresponding latches across the matrices using the same address. The read enable and write enable inputs should also be shared, as we want to read or write an entire byte, not just individual bits. Each matrix will have its own data line, providing access to any bit within the byte.
![1750352384839](image/computer_achitecture_part_2/1750352384839.png)
![1750352420685](image/computer_achitecture_part_2/1750352420685.png)

For example, to write a byte to memory, we input all the bits that make up the byte through the data lines and then activate the write enable signal. This action causes each bit of the byte to be stored in the corresponding latch at the specified address in each matrix. The byte is now stored in memory. Later, if we want to read that byte, we input the address and activate the read enable signal. This causes the data lines to output the value of each bit that makes up the byte at that address.
![1750352451443](image/computer_achitecture_part_2/1750352451443.png)
![1750352536705](image/computer_achitecture_part_2/1750352536705.png)

When we write programs, we don't need to concern ourselves with the underlying electronic matrix etched into the hardware. Instead, we think of bytes as sequences of bits stored contiguously. Thus, we abstract this further and conceptualize memory as a list of bytes. In this abstraction, addresses aren't intersections in an unseen matrix, but rather indices in this list of bytes. This is called Random Access Memory, since we can point to any random byte in the list by providing its address.
![1750352630635](image/computer_achitecture_part_2/1750352630635.png)
![1750353155001](image/computer_achitecture_part_2/1750353155001.png)
![1750353180496](image/computer_achitecture_part_2/1750353180496.png)

To simplify things, we can define these sets of wires as a bus. The width of the address bus determines how many bytes we can handle. For example, with an 8-bit address bus using two decoders, each capable of controlling 16 different outputs, we can locate any of 256 bits within the matrix.
![1750353245857](image/computer_achitecture_part_2/1750353245857.png)

Generally, if the address bus has n inputs, 2 to the power of n bytes can be managed. For instance, a 32-bit wide address bus can address 4 billion bytes, which explains why 32-bit computers can only handle up to 4 GB of RAM.
![1750353335946](image/computer_achitecture_part_2/1750353335946.png)

Finally, keep in mind that the type of memory we have learned about today is known as static RAM. This type of memory is very fast, but all these logic gates require quite a few transistors, so the production cost is higher. It turns out that those small memory cells in the matrix can also be achieved using a single special transistor called a MOSFET and a capacitor. It is cheaper to produce, but it is slower than static RAM. I'll make a video comparing these kinds of memory.
![1750353363740](image/computer_achitecture_part_2/1750353363740.png)
![1750353382845](image/computer_achitecture_part_2/1750353382845.png)

And that about wraps it up for now. In the next episode, we will interconnect all the components we've learned about in this and previous episodes. This will help us understand how instructions like load, move, and store work at the hardware level to make data flow between components.