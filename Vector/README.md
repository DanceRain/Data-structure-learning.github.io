# 说明文档

主要对如下内容进行了说明:

+ deduplicate
+ bubbleSort
+ mergeSort
+ binarySearch



## deduplicate 

```c++
template<typename T>
int Vector<T>::deduplicate()
{
	int oldsize = _size;
	for (int i = 1; i != _size; )
	{
		if (find(_elem[i], 0, i) != -1)
		{
			remove(i);
		}
		else
		{
			++i;
		}
	}
	return oldsize - _size;
}
```

该函数利用find接口，从第一个数开始往前查找，这种删除方式的来源于下面这种原始的方式

```c++
template<typename T>
int Vector<T>::deduplicate()
{
	int oldsize = _size;
	for (int i = 0; i != _size;)
	{
		if (find(_elem[i], i, _size) != -1)
		{
			remove(i);
		}
        else
        {
            ++i;
        }
	}
	return oldsize - _size;
}
```

这两种方式的唯一不同就是一个是从小范围到大范围，一个则是从大范围到小范围。但是我认为前一种方式更为快捷，因为从小范围开始查找，查找到后便删除元素可以使后面逐渐变大的范围查找时间不至于为o(n)，而一开始便从0到n查找,则该轮的查找时间就为o(n)了。



## bubbleSort

```c++
template<typename T>
void Vector<T>::bubbleSort(Rank lo, Rank hi)
{
	while (!bubble(lo, hi));
}
template<typename T>
bool Vector<T>::bubble(Rank lo, Rank hi)
{
	bool judge = true;
	while (++lo < hi)
	{
		if (_elem[lo] < _elem[lo - 1])
		{
			judge = false;
			std::swap(_elem[lo], _elem[lo - 1]);
		}
	}
	return judge;
}
```

我们先来看未经过优化的冒泡排序:

```c++
template<typename T>
void Vector<T>::bubbleSort(Rank lo, Rank hi)
{
    int size = hi - lo;
    for(int i = lo; i != hi - 1; ++i)
    {
        for(int j = 0; j != size - i - 1; ++j)
        {
            if(_elem[j] > _elem[j + 1])
            {
                std::swap(_elem[j], _elem[j+1]);
            }
        }
    }
}
```

为什么说第二种冒泡排序是未经过优化的呢？原因是因为如果向量整体已经有序，第二种方法并不会停止比较，所以，无论如何，该算法的时间复杂度都为O(n^2)。而第一种，会增加一个一个判断变量，用于在一轮比较下来是否存在乱序的元素，如果存在，则当前向量还未有序，如果不存在，则判断变量返回TRUE，整体则是有序的。则停止排序。这种思想所根据的原理是： a > b, b > c 则 a > c;



## mergeSort

```c++
/*归并排序*/
template<typename T>
void Vector<T>::mergeSort(Rank lo, Rank hi)
{
    /*将一个向量分割成两个部分，直到不能再继续分为止。*/
	if (lo == hi - 1)
	{
		return;
	}
	Rank mid = (lo + hi) >> 1;
	mergeSort(lo, mid);
	mergeSort(mid, hi);
    
    /*进行组合*/
	merge(lo, mid, hi);
}

/*组合的方法在于比较两个向量的第一个元素的大小，如果按照升序排序，则将小的那个元素放到待排序的向量的第一个位置。*/
template<typename T>
void Vector<T>::merge(Rank lo, Rank mi, Rank hi)
{
	Rank leftSize = mi - lo;
	Rank rightSize = hi - mi;
	T* A = _elem + lo;

	T* leftElem = new T[leftSize];
	for (int i = 0; i != leftSize; ++i)
	{
		leftElem[i] = A[i];
	}
	T* rightElem = _elem + mi;

	for (int i = 0, j = 0, k = 0; (i < leftSize) || (j < rightSize);)
	{
        /*i < leftSize是指当前向量还有未排序的元素，j < rightSize是指另一个向量的元素都已经排序完毕，如果此时i < leftSize则将该向量中的元素依次插入直到所有元素都已插入到待排向量中去*/
		if ((i < leftSize) && ((leftElem[i] <= rightElem[j]) || !(j < rightSize)))
		{
			A[k++] = leftElem[i++];
		}
		if ((j < rightSize) && ((rightElem[j] <= leftElem[i]) || !(i < leftSize)))
		{
			A[k++] = rightElem[j++];
		}
	}
	delete[]leftElem;
}
```

归并为何能使所有元素都有序。其道理在于，将元素对半分割，直到最小，也就是向量只有一个元素时，此时合并，就会使在所有成对的（如果向量为奇数则最后一个单独成向量自然有序）向量有序，依次网上合并，则所有成四的向量有序。。。。，由此则完成了向量整体的有序。



## binarySearch

```c++
template<typename T>
int Vector<T>::binarySearch(const T& e, Rank lo, Rank hi)
{
	while (lo < hi)
	{
		Rank mid = (lo + hi) >> 1;
		e < _elem[mid] ? hi = mid : lo = mid + 1;
	}
	return --lo;
}
```

binarySearch的十分好懂，无非就是将向量分割，对中间值进行比较，如果比中间值大则往上继续分割，反之，如果比中间值小，则往下分割，直到不能分割为止。

不过在上述的binarySearch过程中，使e >= _elem[mid]时lo = mid + 1，这就使得，找到的那个元素要么是在向量中小于待查元素的最大值，要么是等于该值（如果等于该值的元素有多个，则返回的秩为等于最后那个相等元素的秩）。如果是比向量中最小元素还有小的则，则返回秩-1；

这种返回结果的好处是，如果想要在有序向量插入一个元素且继续保持向量有序性，只需要通过查找要插入的元素返回的秩来进行插入即可。



## 边界问题

关于各种排序查找的边界问题始终是使我头痛的一个问题。例如mergeSort中mid的确认，二分查找中lo和hi的确认。

因此我在这里想要理清楚这些东西

```c++
int BinSearch(SeqList *R，int n,KeyType K)
{
    //在有序表R[0..n-1]中进行二分查找，成功时返回结点的位置，失败时返回-1
    int low=0,high=n-1,mid；     //置当前查找区间上、下界的初值
    while(low<=high)
    {
        if(R[low].key==K)
            return low;
        if(R[high].key==k)
            return high;          //当前查找区间R[low..high]非空
        mid=low+(high-low)/2;
            /*使用(low+high)/2会有整数溢出的问题
            （问题会出现在当low+high的结果大于表达式结果类型所能表示的最大值时，
                这样，产生溢出后再/2是不会产生正确结果的，而low+((high-low)/2)
                不存在这个问题*/
        if(R[mid].key==K)
          return mid;             //查找成功返回
        if(R[mid].key<K)
          low=mid+1;              //继续在R[mid+1..high]中查找
        else
          high=mid-1;             //继续在R[low..mid-1]中查找
    }
    if(low>high)
        return -1;//当low>high时表示所查找区间内没有结果，查找失败
}
```

该代码来自百度的普通binarySearch，这个程序十分复杂，但是不妨我们使用他来进行边界问题分析。

在程序的开头它使用了high = n - 1， 也就是说排序会从第一个元素的秩到最后一个元素的秩之间进行。因此他的循环条件就必须得是low<=high，因此他的low和high就必须使用low=mid+1和high=mid-1这样笨拙的避免在找不到值的情况下而出现死循环的方法。

下面再看另外一种简单的二分查找

```c++
template<typename T>
int Vector<T>::binarySearch(T& A, const T& e, Rank lo, Rank hi)
{
	while (lo + 1 < hi)
	{
		Rank mid = (lo + hi) >> 1;
		(e < A[mid]) ? hi = mid : lo = mid;
	}
	return (e == A[lo]) ? lo : -1;
}
```



那么再来看我们的binarySearch，他是对[lo, hi）秩之间的对应元素进行排序，他相比较上面方法更加简单。首hi<=size是我们要知道的。既然如此，lo + 1< hi我们也就理解了，如果hi - 1 == lo也就说明二分已经到头了，不能再分了，然后退出再比较e是否等于A[lo]。