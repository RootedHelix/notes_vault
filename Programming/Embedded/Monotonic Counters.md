Related [[RingBuffer Mask Pattern]], [[Ring Buffer - Compile Logging Time Relationship]], [[Embedded]]

#embedded #embedded_ring_buffer #ringbuffer_mask_pattern #monotonic_counters

Yes — exactly. 👍

With monotonic counters, the head and tail just keep increasing, and you only convert them to a real buffer index when you access the array.

  

Let’s walk through it clearly.

  

  

  

  

1. The Counters Never Wrap

  

  

Instead of doing wrap-around math on the counters themselves:

head = (head + 1) % size;

you do:

head++;

and leave it alone.

  

Same with the tail:

tail++;

So internally you might have:

head = 1052

tail = 1047

Those numbers represent total writes and total reads since boot.

  

  

  

  

2. Index Into the Buffer Only When Accessing Memory

  

  

The actual array index is calculated when you touch the buffer:

index = head % size;

Example:

  

Buffer size = 8

head = 13

tail = 10

Memory positions:

head_index = 13 % 8 = 5

tail_index = 10 % 8 = 2

Buffer visualization:

[ ][ ][X][X][X][ ][ ][ ]

       ^       ^

      tail    head

But the real number of elements is simply:

head - tail = 3

  

  

  

  

3. Why This Is Nice

  

  

The math becomes extremely simple.

  

  

Elements in buffer

  

count = head - tail;

  

Empty

  

head == tail

  

Full

  

(head - tail) == size

No tricky wrap logic needed.

  

  

  

  

4. Why This Works

  

  

Because unsigned integers naturally wrap around.

  

Example with uint32_t:

max = 4,294,967,295

After that:

head = 0

But subtraction still works correctly due to modular arithmetic.

  

So:

head - tail

still gives the correct number of elements.

  

This trick is widely used in:

  

- OS kernels
- lock-free queues
- high performance networking
- DMA ring descriptors

  

  

  

  

  

5. Visual Comparison

  

  

  

Classic ring buffer

  

indexes wrap

head = (head + 1) & mask

Capacity:

size - 1

  

  

  

  

Monotonic counter buffer

  

head++

tail++

index = counter % size

Capacity:

size

  

  

  

  

6. Why Microcontrollers Often Use Your Version

  

  

Your implementation used:

(head + 1) & MASK

Advantages on MCUs:

  

- extremely fast
- no division/modulo
- smaller compiled code
- deterministic timing

  

  

That’s why it’s common in firmware.

  

  

  

  

7. Where Monotonic Counters Show Up

  

  

This pattern appears in serious systems:

  

- network card descriptor rings
- lock-free queues
- audio processing pipelines
- kernel tracing buffers
- packet processing engines

  

  

Even parts of the Linux kernel networking stack use this style.

  

  

  

✅ Your summary is correct

  

The counters grow forever, and the real buffer index is computed when accessing memory.

  

That’s the core idea.

  

  

  

If you’re interested, there is one more extremely cool evolution of this idea:

lock-free ring buffers used by network cards and multicore systems, where multiple CPUs write and read concurrently without locks.

  

That design is a huge milestone in systems programming.