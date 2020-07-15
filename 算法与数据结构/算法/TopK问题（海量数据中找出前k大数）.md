1. 采用基于小根堆的 优先队列 ，前N个元素输入时，建立大小为N 的小根堆，之后每输入一个元素，便调整堆结构，并将根结点的元素（即最小元素）移除。直到所有元素输入完成后，留在堆里的N个元素便是最大的N 个元素，此时根结点的元素便是第 N大的元素。
>O(Nlogk)
```
public class MinPQ<Key extends Comparable<Key>> {
    private Key[] pq;   //基于堆的完全二叉树
    private int N = 0;  //存储于pq[1..N]，pq[0]不使用

    public MinPQ(int maxN) {
        pq = (Key[]) new Comparable[maxN + 1];
    }
    public boolean isEmpty() {
        return N == 0;
    }
    public int size() {
        return N;
    }
    public void insert(Key v) {
        pq[++N] = v;    //先将新元素添加在堆末尾
        swim(N);        //再上浮到合适位置
    }
    public Key delMin() {
        Key min = pq[1];    //从根节点得到最大元素
        exch(1, N--);       //将其和最后一个结点交换
        pq[N + 1] = null;   //防止对象游离
        sink(1);            //恢复堆的有序性
        return min;
    }
    public void show() {
        for (int i = 1; i <= N; i++) {
            System.out.printf(pq[i] + " ");
        }
        System.out.println();
    }
    private boolean less(int i, int j) {
        return pq[i].compareTo(pq[j]) < 0;
    }
    private void exch(int i, int j) {
        Key t = pq[i];
        pq[i] = pq[j];
        pq[j] = t;
    }
    private void swim(int k) {
        while (k > 1 && less(k, k / 2)) {
            exch(k / 2, k);
            k = k / 2;
        }
    }
    private void sink(int k) {
        while (2 * k <= N) {
            int j = 2 * k;
            if (j < N && less(j + 1, j))
                j++;
            if (less(k, j))
                break;
            exch(k, j);
            k = j;
        }
    }
}
```

```
public static String select(String[] a, int k) {
        MinPQ<String> pq = new MinPQ<>(k + 1);  //多一个用于重排和删除
        for (int i = 0; i < a.length; i++) {
            pq.insert(a[i]);
            if (pq.size() > k)
                pq.delMin();
        }
        return pq.delMin();
    }
```

2. 结合快速排序中的划分方法(partition())和二分法。一次划分后，a[j] 左侧的元素都大于 a[j] ，右侧的都小于 a[j] ，比较 k-1 与 j ，如果相等，则 a[j]便是第 k大的元素，a[0..j] 为前 k 大的元素；若 j>k-1 ，则对 a[j] 左半部分进行划分；若 j<k-1 ，则对 a[j]右半部分进行划分，直到找到第 k大的元素


```
public static Comparable select(Comparable[] a, int k) {
        k --;
        shuffle(a);
        int lo = 0, hi = a.length - 1;
        while (hi > lo) {
            int j = partition(a, lo, hi);
            if (j == k)
                return a[j];
            else if (j > k)
                hi = j - 1;
            else
                lo = j + 1;
        }
        return a[k];
    }

public static int partition(Comparable[] a, int lo, int hi) {
        int i = lo, j = hi + 1;
        Comparable v = a[lo];  //pivot
        while (true) {
            while (less(v, a[++i])){
                if (i == hi)
                    break;
            }
            while (less(a[--j], v)) {
                if (j == lo)
                    break;
            }
            if (i >= j)
                break;
            exch(a, i, j);
        }
        exch(a, lo, j);
        return j;
    }
```
