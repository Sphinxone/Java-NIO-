# Buffer

## 整体分析

Buffer是一个抽象类，Buffer类对于各种基础数据类型的缓冲区都做了顶层抽象。

## 变量解析

     static final Unsafe UNSAFE = Unsafe.getUnsafe();
     static final ScopedMemoryAccess SCOPED_MEMORY_ACCESS = ScopedMemoryAccess.getScopedMemoryAccess();

Unsafe是直接对于内存进行操作的类

    private int mark = -1;
    private int position = 0;
    private int limit;
    private int capacity;

标志，偏移量，限制大小和容量。

标志是对于偏移量的一个记号，通过``mark()``函数设置标志位记录当前的位置。

偏移量，记录当前位置。

限制大小，由于Buffer存在读和写两种模式，那么在写模式下``position``记录当下所需要填充的位置，但是当使用``flip()``函数之后会进入读模式，那么就需要记录当前的偏移量，并从头开始读取数据，limit在此时就会记录当下的``position``。

    public final int limit() {
        return limit;
    }

    public Buffer limit(int newLimit) {
        if (newLimit > capacity | newLimit < 0)
            throw createLimitException(newLimit);
        limit = newLimit;
        if (position > newLimit) position = newLimit;
        if (mark > newLimit) mark = -1;
        return this;
    }

`limit`相关的函数，通过重载实现对于`limit`的读与写。

    public Buffer reset() {
        int m = mark;
        if (m < 0)
            throw new InvalidMarkException();
        position = m;
        return this;
    }

    public Buffer mark() {
        mark = position;
        return this;
    }

`mark()`是对于当前`position`的标记，`reset()`是对于`position`的重启，将其定位到`mark`所在位置。

    public Buffer rewind() {
        position = 0;
        mark = -1;
        return this;
    }

`rewind()`方法对于Buffer进行重置，这里和Array类似，不释放内存，而是直接对于`position`进行重置。

    public final int remaining() {
        int rem = limit - position;
        return rem > 0 ? rem : 0;
    }

    public final boolean hasRemaining() {
        return position < limit;
    }

这里的两个函数与读模式相关，都是查询有多少的数据可以读取，或者是否有数据可以被读取。从这里可以看出`position`和`limit`之间的相互作用。

    public abstract boolean isReadOnly();
    public abstract boolean hasArray();
    public abstract Object array();
    public abstract int arrayOffset();
    public abstract boolean isDirect();
    public abstract Buffer slice();
    public abstract Buffer slice(int index, int length);
    public abstract Buffer duplicate();
    abstract Object base();

这些是与数组相关的方法

    final int nextGetIndex() {                          // package-private
        int p = position;
        if (p >= limit)
            throw new BufferUnderflowException();
        position = p + 1;
        return p;
    }

    final int nextGetIndex(int nb) {                    // package-private
        int p = position;
        if (limit - p < nb)
            throw new BufferUnderflowException();
        position = p + nb;
        return p;
    }

        final int nextPutIndex() {                          // package-private
        int p = position;
        if (p >= limit)
            throw new BufferOverflowException();
        position = p + 1;
        return p;
    }

    final int nextPutIndex(int nb) {                    // package-private
        int p = position;
        if (limit - p < nb)
            throw new BufferOverflowException();
        position = p + nb;
        return p;
    }

获取`position`用于读写

