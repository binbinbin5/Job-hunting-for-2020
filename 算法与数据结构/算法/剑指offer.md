[TOC]
# 剑指OFFER


### 单例模式

```
//懒汉，线程不安全
Singleton{
    priavte static Singleton a = null;
    private Singleton(){}
    
    public static Singleton getSinlgeton(){
        if(a==null){
            a = new Singleton();
        }
        return a;
    }
    
}

//线程安全
public class Singleton {  
    private volatile static Singleton singleton;  
    private Singleton (){}  
    public static Singleton getSingleton() {  
    if (singleton == null) {  
        synchronized (Singleton.class) {  
        if (singleton == null) {  
            singleton = new Singleton();  
        }  
        }  
    }  
    return singleton;  
    }  
}
```


### 数组中包含相同数字
```
public static boolean isHaveSameNumbers(int[] numbers){
        //数组中是否包含相同数字
        if(numbers.length<=1){
            return false;
        }

        HashMap<Integer,Integer> tem = new HashMap<>();
        for (int i = 0; i < numbers.length; i++) {
            if(tem.containsKey(numbers[i])){
                return true;
            }else {
                tem.put(numbers[i],i);
            }

        }
        return false;
    }
```
### 返回数组中所有重复值
```
public static List<Integer> findSameNumber(int[] numbers){
        //返回重复的值
        if(numbers.length<=1){
            return null;
        }

        List a = new ArrayList();
        HashMap<Integer,Integer> tem = new HashMap<>();
        for (int i = 0; i < numbers.length; i++) {
            if(tem.containsKey(numbers[i])){
                a.add(numbers[i]);

            }else {
                tem.put(numbers[i],0);
            }

        }
        return a;
    }
```
### 矩阵查找数
```
/**
* 在一个二维数组中，每一行都按照从左到右递增
* 的顺序排序，每一列都按照从上到下递增的顺序排序。
* 请完成一个函数，输入这样的一个二维数组
* 和一个整数，判断数组中是否函数该整数。
*/
public static boolean isHaveSameNumberInTwo(int target,int[][] array){
        //target是否再array中
        //按照区域来

        int r = array[0].length;//列
        int l = array.length;//行

        int s = 0;//开始行
        int e = r-1;
        while(s<l&&e>=0){//行小与最大行，列大于0，从右上开始比较
            if(array[s][e]>target){//不在这一列
                //列--
                e--;
            }else if(array[s][e]<target){//可能在这列
                s++;
            }else {
                return true;
            }
        }
        return false;
    }
```
### 空格替换
```
 /**
         * 请实现一个函数，把字符串中的每个空格替换成"%20"。
         * 例如输入"We are happy"，则输出"We%20are%20happy"
         */
 public static String replaceBlank(char[] charArray){
       
        //char[] charArray = str.toString().toCharArray();
        List<Character> charReplce = new ArrayList();
        for (int i =0;i<charArray.length;i++){
            if(charArray[i] == ' '){
                charReplce.add('%');
                charReplce.add('2');
                charReplce.add('0');
            }else{
                charReplce.add(charArray[i]);
            }
        }
        char[] b = new char[charReplce.size()];
        for (int i = 0; i <charReplce.size() ; i++) {
            b[i] = charReplce.get(i);
        }
        return new String(b);
    }
```
### 有序数组合并
```
// AB两个有序数组合并排序，A的空间足够大
// a[4] ={1,3} b[2] = {2,4} 
 public static void mergeSortedArray(int A[], int m, int B[], int n) {
        int total = m+n;
        while(m > 0 && n > 0){
            if(A[m-1] < B[n-1]){
                A[total - 1] = B[n - 1];
                n--;
            }
            else{
                A[total - 1] = A[m - 1];
                m--;
            }
            total--;
        }
        //A先遍历完，因为里面有很多empty
        while(n > 0){
            A[total - 1] = B[n - 1];
            total--;
            n--;
        }
    }
```
### 反向打印链表
```
public class Node {
    public int val;
    Node next;
    Node(int val){
        this.val = val;
        next = null;
    }
}
 //从头到尾打印链表
public static List<Integer> printLinkedRerver(Node listNode){
    ArrayList<Integer> printNode = new ArrayList<>();
    while (listNode!=null){
        printNode.add(listNode.val) ;
        listNode = listNode.next;
    }
    ArrayList<Integer> printNode1 = new ArrayList<>();
    for (int i = printNode.size()-1; i >=0 ; i--) {
        printNode1.add(printNode.get(i));
    }

    return printNode1;
}
//反转打印链表  
public static Node reverseNode(Node a){
        
        Node pre = null;
        Node end = null;
        while (a!=null){
            end = a.next;
            a.next = pre;
            pre = a;
            a = end;
        }
        end = pre;
        while (pre!=null){
            System.out.print(pre.val);
            pre = pre.next;
        }
        return end;
    }    
    
```
### 重建树(*)
```
public class NodeTree {
    int val;
    NodeTree left;
    NodeTree right;
    NodeTree(int x) { val = x; }
}
//重建二叉树，输入前序中序
//1.前序第一个得到根，中序可以以根左是左子树，右是右子树
//2.1中的左开始，一样的方式 得到左右
//3.重复步骤
public static NodeTree reBuildTree(int[] pre,int[] in){
    NodeTree root=reConstructBinaryTree(pre,0,pre.length-1,in,0,in.length-1);
    return root;
}

private static NodeTree reConstructBinaryTree(int [] pre,int startPre,int endPre,int [] in,int startIn,int endIn) {

    if(startPre>endPre||startIn>endIn)
        return null;

    NodeTree root=new NodeTree(pre[startPre]);

    for(int i=startIn;i<=endIn;i++)
        if(in[i]==pre[startPre]){
            //startPre+i-startIn中（i-strarIn为左个数）
            //前序的右子树开始就是前序左子树的位置+1 
            root.left=reConstructBinaryTree(pre,startPre+1,startPre+i-startIn,in,startIn,i-1);//前中序的左
            root.right=reConstructBinaryTree(pre,i-startIn+startPre+1,endPre,in,i+1,endIn);//前中序的右
            break;
        }

    return root;
}

// 将二叉树先序遍历，用于测试结果
public static void preTraverseBinTree(TreeNode node) {
	if (node == null) {
		return;
	}
	System.out.print(node.val + " ");
	if (node.left != null) {
		preTraverseBinTree(node.left);
	}
	if (node.right != null) {
		preTraverseBinTree(node.right);
	}
}

// 将二叉树中序遍历，用于测试结果
public static void inTraverseBinTree(TreeNode node) {
	if (node == null) {
		return;
	}
	if (node.left != null) {
		inTraverseBinTree(node.left);
	}
	System.out.print(node.val + " ");
	if (node.right != null) {
		inTraverseBinTree(node.right);
	}
}

// 将二叉树后序遍历，用于测试结果
public static void postTraverseBinTree(TreeNode node) {
	if (node == null) {
		return;
	}
	if (node.left != null) {
		postTraverseBinTree(node.left);
	}
	if (node.right != null) {
		postTraverseBinTree(node.right);
	}
	System.out.print(node.val + " ");
}

```
### 旋转数据查找（*）
```
//旋转数组查找元素：二分法
public int minNumberInRotateArray(int [] array) {
    int len = array.length;
    if(len == 0)
        return 0;
    int low = 0, high = len - 1;
    while(low < high){
        int mid = low + (high - low) / 2;
        if(array[mid] > array[high]){ //中位数大于高位，旋转点在中右边
            low = mid + 1;//最小值肯定在右边
        }else if(array[mid] == array[high]){ //相同缩小范围
            high = high - 1;
        }else{//小于 旋转点在左边，直接查左边
            high = mid;
        }
    }
    return array[low];
}
```

```
//旋转数据查找目标值
public static boolean rotatedBinarySearch(int[] arr,int target) {
    int start = 0, end = arr.length - 1;
    while (start <= end) {
        int mid = start + (end - start) / 2;
        if (arr[mid] == target) { 
            return true;
        } else if (arr[mid] > arr[start]) {//情况A：旋转点在中位数右侧
            //比较目标值
            if (arr[mid] > target && arr[start] <= target) {
                end = mid - 1;
            } else {
                start = mid + 1;
            }
        } else {//情况B：旋转点在中位数左侧，或与中位数重合
            //比较目标值
            if (arr[mid] < target && target <= arr[end]) {
                start = mid + 1;
            } else {
                end = mid - 1;
            }
        }
    }
    return false;
}
```
### 两栈成队列
```
//两栈成队列
public class MyList {
    Stack<Integer> stack1 = new Stack<Integer>();
    Stack<Integer> stack2 = new Stack<Integer>();

    private static int size=0;

    public void push(int node) {
        stack1.push(node);
        size++;
    }

    public int pop() {

        if(size == 0){

                try {
                    throw new Exception("队列为空");
                } catch (Exception e) {
                    e.printStackTrace();
                }

        }
        for (int i = 0; i < size-1; i++) {
            stack2.push(stack1.pop());
        }

        int temp = stack1.pop();
        size--;
        for (int i = 0; i < size; i++) {
            stack1.push(stack2.pop());
        }


        return temp;
    }

}

```


### 斐波那契
```
//斐波那契
//递归：利用备忘录记录 减少计算时间

static HashMap<Interger,Interger> men = new HashMap<>():
public static int Fibonacci(int n){
        if(!men.containsKey(n)){
             men.put(n,n<2?n:(Fibonacci(n-2)+Fibonacci(n-1)));
        }
        return men.get(n);
}
```
```
//迭代    
private static void ThreeVariable(int n) {

    if(n == 0 || n == 1){
       return n; 
    }
          
    int fn1 = 0;
    int fn2 = 1;
    for(int i=2; i<=n; i++){
        fn2 += fn1;
        fn1 = fn2 - fn1;
    }
    return fn2;
}  

```

```
//一只青蛙一次可以跳上1级台阶，也可以跳上2级。求该青蛙跳上一个n级的台阶总共有多少种跳法（先后次序不同算不同的结果）。 就是斐波那契。

//直接计算
public static int JumpFloor(int target){
    if(target<0){
        return 0;
    }
    else if(target<=2){
        return target;
    }else {
    //站在第N阶上，数量 = （N-1） + （N-2） 
        return  (JumpFloor(target-1)+JumpFloor(target-2));
    }
}
//备忘录
static HashMap<Interger,Interger> men = new HashMap<>():
public static int JumpFloor(int n){
        if(!men.containsKey(n)){
             men.put(n,n<2?n:(JumpFloor(n-2)+JumpFloor(n-1)));
        }
        return men.get(n);
}


```

```
//迭代
public static int iterJump(int n){
    if(n<1){
        return 0;
    }else if(n<3){
        return n;//1,2层
    }

    int a = 1;//一层
    int b = 2;//两层
    int sum = 0;//N层
    for (int i = 2; i < n; i++) {
        //每次循环相当于 前两次相加
        //比如 3层，那么 1层+2层
        sum= a+b;
        a = b;
        b = sum;
    }
    return sum;
}
```

```
//一只青蛙一次可以跳上1级台阶，也可以跳上2级……它也可以跳上n级。求该青蛙跳上一个n级的台阶总共有多少种跳法
//f(n)= f(n-1)+f(n-2)+...f(1)
// 1.2.4.8.16.32.64....

public static int JumpFloorII(int target) {
    if(target<=0) return 0;
    return (target == 1) ? 1 : 2*JumpFloorII(target-1);

}

```

```
//我们可以用2*1的小矩形横着或者竖着去覆盖更大的矩形。请问用n个2*1的小矩形无重叠地覆盖一个2*n的大矩形，总共有多少种方法？
//和走台阶一样 f(n) = f(n-1)+f(n-2) 这边只列出基本解决办法

//迭代
public static int RectCover(int n){
    if(n<1){
        return 0;
    }else if(n<3){
        return n;
    }

    int a = 1;
    int b = 2;
    int sum = 0;
    for (int i = 2; i < n; i++) {
        sum= a+b;
        a = b;
        b = sum;
    }
    return sum;
}
```
### 矩阵走路(*)
```
//矩阵可以从任意一点上下左右走出一条路，符合某个字符串
// 如果一条路径经过了矩阵中的某一个格子，则之后不能再次进入这个格子。
//1. 从矩阵中找出字符串头数据
//2. 从该点分别上下左右（回溯）寻找字符串第二个数，若找不到返回1继续找头结点
//3. 若找到则从第三个点开始找，找不到则回溯
//4. 设置一个矩阵记录每个格子是否走过，走过就不能走了


public boolean hasPath(char[] matrix, int rows, int cols, char[] str) {
    if (matrix == null || rows < 1 || cols < 1 || str == null) {
        return false;
    }
    boolean[] isVisited = new boolean[rows * cols];
    for (boolean v : isVisited) {//记录是否走过
        v = false; 
    }
    int pathLength = 0;
    for (int row = 0; row < rows; row++) {
        for (int col = 0; col < cols; col++) {
            if (hasPathCore(matrix, rows, cols, row, col, str, pathLength, isVisited))
                return true;
        }
    }
    return false;
}

private boolean hasPathCore(char[] matrix, int rows, int cols, int row, int col, char[] str, int pathLength,
                            boolean[] isVisited) {
    if (row < 0 || col < 0 || row >= rows || col >= cols || isVisited[row * cols + col] == true
            || str[pathLength] != matrix[row * cols + col])
        return false;
    
    if (pathLength == str.length - 1)//长度一样代表找到了
        return true;
   
    boolean hasPath = false;
    isVisited[row * cols + col] = true; //记录这点走了
    hasPath = hasPathCore(matrix, rows, cols, row - 1, col, str, pathLength + 1, isVisited) //上下左右是否能走通
            || hasPathCore(matrix, rows, cols, row + 1, col, str, pathLength + 1, isVisited)
            || hasPathCore(matrix, rows, cols, row, col - 1, str, pathLength + 1, isVisited)
            || hasPathCore(matrix, rows, cols, row, col + 1, str, pathLength + 1, isVisited);

    if (!hasPath) {//走不通 这个点恢复
        isVisited[row * cols + col] = false;
    }
    return hasPath;
}

```
### 机器人走路（*）
地上有一个m行和n列的方格。一个机器人从坐标0,0的格子开始移动，每一次只能向左，右，上，下四个方向移动一格，但是不能进入行坐标和列坐标的数位之和大于k的格子。 例如，当k为18时，机器人能够进入方格（35,37），因为3+5+3+7 = 18。但是，它不能进入方格（35,38），因为3+5+3+8 = 19。请问该机器人能够达到多少个格子？
```
//1.使用一个访问数组记录是否已经经过该格子。
//2.机器人从(0,0)开始移动，当它准备进入(i,j)的格子时，通过检查坐标的数位来判断机器人是否能够进入。
//3.如果机器人能进入(i,j)的格子，接着在判断它是否能进入四个相邻的格子(i,j-1),(i,j+1),(i-1,j),(i+1,j)。
//4.因此，可以用回溯法来解决这一问题。

 //回溯法
public int movingCount(int threshold, int rows, int cols) {
    if(rows<=0||cols<=0||threshold<0)   return 0;
    int[] visited=new int[rows*cols];//记录点是否走过
    return MovingCount(threshold,rows,cols,0,0,visited);
}

private int MovingCount(int threshold,int rows,int cols,int row,int col,int[] visited){
    int count=0;
    if(canWalkInto(threshold, rows, cols, row, col, visited)){ //这点能走
        visited[row*cols+col]=1;//记录改点
        //判断四周能走不
        count=1+MovingCount(threshold,rows,cols,row-1,col,visited)   //往上
                +MovingCount(threshold,rows,cols,row+1,col,visited)  //往下
                +MovingCount(threshold, rows, cols, row, col-1, visited)   //往左
                +MovingCount(threshold, rows, cols, row, col+1, visited);  //往右
    }
    return count;
}

private boolean canWalkInto(int threshold,int rows,int cols,int row,int col,int[] visited){ //判断能走不
    if(row>=0 && row<rows && col>=0 && col<cols
            && getSumOfDigits(row)+getSumOfDigits(col)<=threshold
            && visited[row*cols+col]==0)
        return true;
    else
        return false;
}

private int getSumOfDigits(int number){ //判断条件
    int sum=0;
    while(number!=0){
        sum+=number%10;
        number/=10;
    }
    return sum;
}


```
### 二进制计算1
```
//判断n的二进制有几个1，正负按照补码算
public static  int er(int n){
    int count = 0;
    while(n!=0){
        count += n&1; //n&1判断最后一个是不是1 也相当于 n%2 ==1
        n=n>>>1;//无符号右移
    }
    return count;

}
```
### 数值的次方
```
//可以使用右移运算符代替除以2，用位与运算符代替求余运算符（%）来判断一个数是奇数还是偶数。
//递归熟悉的可以考虑： logN复杂度
public double PowerUnsignedExponent(double base, int n){
        if(n == 0)
            return 1;
        if(n == 1)
            return base;
        //递归
        double res = PowerUnsignedExponent(base, n>>1);
        res *= res;
        if((n & 0x1) == 1)//奇数
            res *= base;
        return res;
}
```

```
//时间复杂读N 不推荐 一般人都能做出来
 public double Power(double base, int exponent) {
        double sum = 1.0;
        if(exponent==0){
            return 1;//考虑0^0=1.0边界值
        }else if(exponent==1){
            return base;
        }else if(exponent>1){//指数大于0
            for(int i=0;i<exponent;i++){
                sum=base*sum;
            }
            return sum;
        }else{//指数小于0
            exponent=-(exponent);
            for(int i=0;i<exponent-1;i++){
                sum=base*sum;
            }        
            return (1/sum);
        }
  }
```


```
//上两种的思想结合
//logN的时间复杂度，也没有递归
public static double fun(double val,int n){
        boolean isF = false;
        if(n==1){
            return val;
        }else if(n==0){
            return 1;//val=0的情况记得考虑
        }else if(n<0){
            n = -(n);
            isF = true;
        }

        // 2^N = 2^(N/2) * 2^(N/2) //偶数 2为底数
        // 2^N = 2^((N-1)/2) * 2^((N-1)/2) *2 //奇数
        double res = 1;
        while(n != 0)
        {
            if((n & 1) == 1)
            {
                res = res * val;
            }
            n >>= 1;
            val = val * val;//重点
        }
        
        if (!isF){
            return res;
        }else {
            return (1/res);
        }
    }
```

### 数组奇在偶前

```
//算法复杂度N，需要格外空间
ArrayList<Integer> ji = new ArrayList<>();
        ArrayList<Integer> ou = new ArrayList<>();
        for (int i = 0; i < array.length; i++) {
            if((array[i]&1) == 1){
                ji.add(array[i]);
            }else {
                ou.add(array[i]);
            }
        }
        ji.addAll(ou);
        for (int i = 0; i < ji.size(); i++) {
            array[i] = ji.get(i);
        }
```

```
//算法复杂度N^2,不需要额外空间
//冒泡
 public void reOrderArray(int [] array) {
        for(int i=0; i < array.length; i++){
            for(int j=0; j<array.length-1; j++){
                if(array[j] % 2 == 0 && array[j+1] % 2 != 0){
                    int temp = array[j+1];
                    array[j+1] = array[j];
                    array[j] = temp;
                }
            }
        }
    }
```
### 链表中倒数第k个结点

```
//快慢指针
  public ListNode FindKthToTail(ListNode head,int k) {
       if(head == null || k == 0)//边界条件
            return null;
        ListNode pre = head;
        ListNode tail = head;
        for (int i = 0; i < k; i++) {
            if(tail.next != null)
                tail = tail.next;
            else
                return null;//放置K大于链表
        }

        while(tail!=null) {
            pre = pre.next;
            tail = tail.next;
        }
        return pre;
    }
```
### 反转链表

```
   public static ListNode ReverseList(ListNode head) {
        ListNode pre = null;
        ListNode temp = null;
        while(head!=null){
            temp = head.next;
            head.next = pre;
            pre = head;
            head = temp;
        }
        return pre;
    }
```

### 合并有序链表


```
  if (list1==null){
            return list2;
        }else if(list2 == null){
            return list1;
        }

        ListNode temp = null;
        if(list1.val>list2.val){
            temp = list2;
            list2 = list2.next;
        }else {
            temp = list1;
            list1 = list1.next;
        }
        ListNode res = temp;
        while (list1!=null&&list2!=null){
            if (list1.val>list2.val){
                temp.next = list2;
                list2 = list2.next;
                temp = temp.next;
            }else {
                temp.next = list1;
                list1 = list1.next;
                temp = temp.next;
            }
        }

        if(list1!=null){
            temp.next = list1;
        }else if(list2!=null){
            temp.next=list2;
        }
        return res;
```
### 判断是否为子树

```
//递归
 public boolean HasSubtree(TreeNode root1,TreeNode root2) {
        if(root1 == null || root2 == null){
            return false;
        }
        return IsSubtree(root1, root2) ||
                HasSubtree(root1.left, root2) ||//根节点不行就换左，再换右（短路原则）
                HasSubtree(root1.right, root2);
    }
    public boolean IsSubtree(TreeNode root1, TreeNode root2){
        //要先判断roo2, 不然{8,8,7,9,2,#,#,#,#,4,7},{8,9,2}这个测试用例通不过。
        if(root2 == null)
            return true;//完成匹配
        if(root1 == null)
            return false;//缺少
        if(root1.val == root2.val){
            return IsSubtree(root1.left, root2.left) &&
                    IsSubtree(root1.right, root2.right);
        }else
            return false;
    }
```
### 二叉树的镜像

```
    源二叉树 
        8
       /  \
      6   10
     / \  / \
    5  7 9 11
    镜像二叉树
        8
       /  \
      10   6
     / \  / \
    11 9 7  5
``` 
```    
 public static void Mirror(TreeNode root) {
        if((root == null)||(root.left==null&&root.right==null))
            return;
        swapTree(root);
        if(root.left!=null){
            Mirror(root.left);
        }
        if(root.right!=null){
            Mirror(root.right);
        }
    }
    private static void swapTree(TreeNode root){
        TreeNode temp = root.left;
        root.left = root.right;
        root.right = temp;
    }

```
### 顺时针旋转矩阵

```
public static ArrayList<Integer> printMatrix(int [][] matrix) {
        int row = matrix.length;//hang
        int col = matrix[0].length;//lie
        ArrayList<Integer> res = new ArrayList<>();

        if(row == 0 && col == 0)
            return res;
        int left = 0, right = col - 1, top = 0, bottom = row - 1;
        while(left <= right && top <= bottom){
            //上：从左到右
            for(int i=left; i<=right; i++)
                res.add(matrix[top][i]);
            //右：从上到下
            for(int i=top+1; i<=bottom; i++)
                res.add(matrix[i][right]);
            //下：从右到左
            if(top != bottom){
                //防止单行情况
                for(int i=right-1; i>=left; i--)
                    res.add(matrix[bottom][i]);
            }
            //左：从下到上
            if(left != right){
                //防止单列情况
                for(int i=bottom-1; i>top; i--)
                    res.add(matrix[i][left]);
            }
            left++; right--; top++; bottom--;
        }
        return res;}
```


```
public static ArrayList<Integer> printMatrix(int [][] matrix) {
        ArrayList<Integer> res = new ArrayList<>();

        int left = 0;
        int top = 0;
        int bottom = matrix.length;//列
        int right = matrix[0].length;//行

        while(left<right&&top<bottom){
            //从左到右
            int i = left;
            while (right>i){
                res.add(matrix[top][i]);
                i++;
            }
            //从上到下
            int j = top+1;
            while (bottom>j){
                res.add(matrix[j][right-1]);
                j++;
            }

            //防止单行
           
            if(top!=(bottom-1)){
                //从右到左
                i = right-2;
                while (left<=i){
                    res.add(matrix[bottom-1][i]);
                    i--;
                }
            }
             if((right-1)!=left){
                j = bottom-2;
                while (j>top){
                    res.add(matrix[j][left]);
                    j--;
                }
            }
          
            left++; right--; top++; bottom--;
        }
        return res;}
```
### 求出栈的min数

```
import java.util.Stack;

public class Solution {
    Stack<Integer> stack = new Stack<>();
    Stack<Integer> temp = new Stack<>();
    int min = Integer.MAX_VALUE;
    public void push(int node) {
        stack.push(node);
        if(node < min){
            temp.push(node);
            min = node;
        }
        else
            temp.push(min);
    }

    public void pop() {
        stack.pop();
        temp.pop();
    }

    public int top() {
        int t = stack.pop();
        stack.push(t);
        return t;
    }

    public int min() {
        int m = temp.pop();
        temp.push(m);
        return m;
    }
}
```
### 栈的出栈顺序

```
遍历压栈顺序，先将第一个放入栈中，这里是1，然后判断栈顶元素是不是出栈顺序的第一个元素，这里是4，很显然1≠4，所以我们继续压栈，直到相等以后开始出栈，出栈一个元素，则将出栈顺序向后移动一位，直到不相等，这样循环等压栈顺序遍历完成，如果辅助栈还不为空，说明弹出序列不是该栈的弹出顺序。

首先1入辅助栈，此时栈顶1≠4，继续入栈2

此时栈顶2≠4，继续入栈3

此时栈顶3≠4，继续入栈4

此时栈顶4＝4，出栈4，弹出序列的位置向后移一位，此时为5，,辅助栈里面是1,2,3

此时栈顶3≠5，继续入栈5

此时栈顶5=5，出栈5,弹出序列向后一位，此时为3，,辅助栈里面是1,2,3
```

```

import java.util.Stack;

public boolean IsPopOrder(int [] pushA,int [] popA) {
    if(pushA.length==0||popA.length==0||popA.length!=pushA.length)//边界值
        return false;
    
    Stack<Integer>stack=new Stack<Integer>();
    
    int j=0;
    for(int i=0;i<pushA.length;i++){
        stack.push(pushA[i]);
        while(!stack.isEmpty()&&stack.peek()==popA[j]){
            stack.pop();
            j++;
        }
    }
    return stack.isEmpty();
}
  
```
### 层序遍历


```
//层序遍历
1.对于不为空的结点，先把该结点加入到队列中
2.从队中拿出结点，如果该结点的左右结点不为空，就分别把左右结点加入到队列中
3.重复以上操作直到队列为空

public ArrayList<Integer> levelOrder(BinaryTreeNode root){
    ArrayList<Integer> res = new ArrayList<>();
    if(root == null)
        return res;
    //Queue is abstract; 不能用 new Queue<TreeNode>();
    Queue<TreeNode> queue = new LinkedList<TreeNode>();
    queue.add(root);
    while(queue.size() != 0){
        root = queue.remove();
        res.add(root.val);//key
        if(root.left != null){
            queue.add(root.left);
        }
        if(root.right != null){
            queue.add(root.right);
        }
    }
    return res;    
}
```


### 二叉查找数判断后续遍历正确与否

```
//左都比小，右都比根大
public static boolean VerifySquenceOfBST(int [] sequence) {
    if(sequence.length == 0){
        return false;
    }
    if(sequence.length == 1){
        return true;
    }

    return judge(sequence,0,sequence.length-1);
}

private static boolean judge(int[] sequence,int start,int root){
    if(start >= root){//终止条件 叶子节点
        return true;
    }
    //先找左右
    int i = start;
    while (sequence[i]<sequence[root]&&i<root){//找出左右界点
        i++;//左边都比根小
    }
    //start~i-1为左，i~root-1为右
    for (int j = i; j < root; j++) {
        if(sequence[j]<sequence[root]){//右边都比根大
             return false;
        }
    }
    return (judge(sequence,start,i-1))&&(judge(sequence,i,root-1));
}
```
### 二叉树路径和为目标值

回溯+DFS

```java
ArrayList<ArrayList<Integer> > res = new ArrayList<ArrayList<Integer> >();
ArrayList<Integer> temp = new ArrayList<Integer>();
public ArrayList<ArrayList<Integer>> FindPath(TreeNode root,int target) {
    if(root == null)//到低
        return res;
    target -= root.val;
    temp.add(root.val);
    if(target == 0 && root.left == null && root.right == null)//到达一条路径
        res.add(new ArrayList<Integer>(temp));
    else{
        FindPath(root.left, target);//往下DFS
        FindPath(root.right, target);
    }
    temp.remove(temp.size()-1);//回溯：确保返回父结点时路径刚好是从根结点到父结点的路径。
    return res;
}        
```
### 复制复杂链表

```
public RandomListNode Clone(RandomListNode pHead)
{
    if(pHead==null)
        return null;
    
    //第一步：复制链表结点
    RandomListNode oldnode=pHead;
    while(oldnode)
    {
        RandomListNode newnode = new RandomListNode(oldnode->label);//用老节点的值申请一个新节点
        newnode.next=oldnode.next;//将新节点的next链上老节点的next
        oldnode.next=newnode;//将新节点插在老节点后面
        oldnode=oldnode.next.next;//由于已经插入了新节点，因此遍历下一个老节点得走2个next
    }
    
    //第二步：置信结点的random指针
    oldnode=pHead;//oldnode指向链表的开始
    while(oldnode)
    {
        RandomListNode newnode=oldnode.next;//由于上面的复制，新节点紧跟在与其对应的老节点后面
        if(oldnode.random!=nullptr)
        {
            //oldnode.random是老节点的random域，再指向next就是该老节点对应的新节点应该指向的对应的random域
            newnode.random=oldnode.random.next;
        }
        oldnode=oldnode.next.next;//只走老节点(为其对应的新节点链上对应的randoum域)
    }
    
    //拆分新/老链表
    oldnode=pHead;//让oldnode指向链表开始
    RandomListNode newHead=oldnode.next;//新链表的头是老链表头的下一个(之前复制了的)
    
    while(oldnode)
    {
        RandomListNode newnode=oldnode.next;//保证newnode一直紧跟咋oldnode后面(老、新节点相对位置)
        oldnode->next=newnode.next;//恢复老节点的next指向
        
        if(oldnode.next!=nullptr)
        {
            newnode.next=oldnode.next.next;//恢复新节点的next指向
        }
        oldnode=oldnode.next;//由于oldnode的next域上面已经恢复了，因此下一个老节点就是oldnode->next
    }
  return newHead;
    
}
```
### 二叉搜索树转成排序的双向链表

输入一棵二叉搜索树，将该二叉搜索树转换成一个排序的双向链表。要求不能创建任何新的结点，只能调整树中结点指针的指向。

```
//O（N）
 public TreeNode Convert(TreeNode pRootOfTree) {
        if(pRootOfTree == null){
           return null;
        }else if(pRootOfTree.left==null&&pRootOfTree.right==null){//假如只有一个根节点，则返回根节点
           return pRootOfTree;
        }
        //中序遍历
        ArrayList<TreeNode> res = new ArrayList<>();
        midSearch(pRootOfTree,res);
        
        //节点重置
        res.get(0).left = null;
        res.get(0).right = res.get(1);
        for (int i = 1; i < res.size()-1; i++) {
            
            res.get(i).left = res.get(i-1);
            res.get(i).right = res.get(i+1);
        }
        res.get(res.size()-1).left =res.get(res.size()-2);
        res.get(res.size()-1).right = null;
        pRootOfTree = res.get(0);
        
        return pRootOfTree;
    }

    public void midSearch(TreeNode root,ArrayList<TreeNode> res){
        if(root== null){
            return;
        }
        midSearch(root.left,res);
        res.add(root);
        midSearch(root.right,res);

    }
```

```
//官方递归版本,推荐
public TreeNode Convert(TreeNode pRootOfTree) {
    if(root==null){//假如根节点为空，返回空
        return null;
    }
    if(root.left==null&&root.right==null){//假如只有一个根节点，则返回根节点
        return root;
    }
    //1、将左子树构造成双链表，并返回该链表头结点left
    TreeNode left=Convert(root.left);
 
    //2、定位到左子树链表的最后一个节点（左子树最右边的节点）
    TreeNode p=left;//创建一个临时节点P,用来遍历找到左链表的最后一个节点(左子树最右边的节点)，p初始化指向做左子树的根节点，
    while(p!=null&&p.right!=null){
        p=p.right;//最终p为左子树最右边的节点
    }
    //3、如果左子树链表不为空，将当前root追加到左子树链表后
    if(left!=null){//左子树链表不为空
        p.right=root;//左子树链表的最后一个节点p（左子树最右边节点）的右指针指向当前root节点
        root.left=p;//当前root节点的左指针指向左子树链表的最后一个节点p（左子树最右边节点）
    }
    //4、将右子树构造成双链表，并返回该链表的头结点right
    TreeNode right=Convert(root.right);
 
    //5、如果右子树链表不为空，将右子树链表追加到当前root后
    if(right!=null){//右子树链表不为空
        right.left=root;//右子树链表的头结点right的左指针指向当前root
        root.right=right;//当前root的右指针指向右子树链表的头结点right
    }
    return left!=null?left:root;//根据左子树链表是否为空返回整个双向链表的头指针。
}
```

```
//非递归实现官方做法
import java.util.Stack;
public TreeNode ConvertBSTToBiList(TreeNode root){
    if(root==null){
        return null;
    }
    Stack<TreeNode> stack=new Stack<TreeNode>();
    TreeNode p=root;//p为临时节点用来遍历树的节点，初始值为根节点root
    TreeNode rootList=null;//用于记录双向链表的头结点
    TreeNode pre=null;//用于保存中序遍历序列的上一个节点，即p的上一个节点
    boolean isFirst=true;//判断是否为左子树链表的第一个节点
    while(p!=null||!stack.isEmpty()){
        while(p!=null){//把左节点放入栈中
            stack.push(p);
            p=p.left;
        }
        p=stack.pop();//此时的p为左子树最左边的节点，
        if(isFirst){//假如是左子树链表的第一个节点
            rootList=p;//将p赋值给root节点
            pre=rootList;
            isFirst=false;
        }else{
            pre.right=p;
            p.left=pre;
            pre=p;
        }
        p=p.right;
    }
    return rootList;
}
```
### 字符串排列(**)
输入一个字符串,按字典序打印出该字符串中字符的所有排列。例如输入字符串abc,则打印出由字符a,b,c所能排列出来的所有字符串abc,acb,bac,bca,cab和cba。
```
//1、求得所有可能出现在第一个位置上的字符，将第一个位子上的字符与后面的交换
//2、固定第一个字符，求后面字符的排列，就涉及递归的问题了

public static ArrayList<String> Permutation(String str) {
    List<String> res = new ArrayList<>();
    if (str != null && str.length() > 0) {
        PermutationHelper(str.toCharArray(), 0, res);
        Collections.sort(res);
    }
    return (ArrayList)res;
}

public static void PermutationHelper(char[] cs, int i, List<String> list) {
    if (i == cs.length - 1) {//最后一位
        String val = String.valueOf(cs);
        if (!list.contains(val))
            list.add(val);
    } else {
        for (int j = i; j < cs.length; j++) {
            swap(cs, i, j);
            PermutationHelper(cs, i+1, list);
            swap(cs, i, j);
        }
    }
}
```
### 数组中出现次数超过一半的数
数组中有一个数字出现的次数超过数组长度的一半，请找出这个数字。例如输入一个长度为9的数组{1,2,3,2,2,2,5,4,2}。由于数字2在数组中出现了5次，超过数组长度的一半，因此输出2。如果不存在则输出0。
```
public static int MoreThanHalfNum_Solution(int [] array) {
    HashMap<Integer,Integer> res = new HashMap<>();

    for (int i = 0; i < array.length; i++) {
        if(res.containsKey(array[i])){
            res.replace(array[i],res.get(array[i])+1);
        }else{
            res.put(array[i],1);
        }
    }
    

    for(Integer key:res.keySet())
    {
        if (res.get(key)*2>len){
            return key;
        }
    }
    return 0;
}
```
### 最小的K个数（快排和最大最小堆）
输入n个整数，找出其中最小的K个数。例如输入4,5,1,6,2,7,3,8这8个数字，则最小的4个数字是1,2,3,4,。

- 法1：先对数组排序，然后取出前k个
- 法2：最小堆
```
 public ArrayList<Integer> GetLeastNumbers_Solution(int [] input, int k) {
         ArrayList<Integer> res = new ArrayList<>();
        if(input == null || k ==0 || k > input.length)
            return res;
       
        Arrays.sort(input);

        for (int i = 0; i < k; i++) {
            res.add(input[i]);
        }
        return res;
    }

```

```
public ArrayList<Integer> GetLeastNumbers_Solution(int [] input, int k) {
    ArrayList<Integer> res = new ArrayList<Integer>();
    if(input == null || k ==0 || k > input.length)
        return res;
        //构建堆
    PriorityQueue<Integer> maxHeap = new PriorityQueue<Integer>(k, new Comparator<Integer>() { 
        public int compare(Integer e1, Integer e2) {
            return e2 - e1;
        }
    });
    
    for(int i=0; i<input.length; i++){
        if(maxHeap.size() != k)//取K个
            maxHeap.offer(input[i]);
        else{
            if(maxHeap.peek() > input[i]){//比较
                maxHeap.poll();
                maxHeap.offer(input[i]);
            }
        }
    }
    
    for(Integer i: maxHeap){
        res.add(i);
    }
    return res;
}
```
### 连续子数组最大和
关键在于，前面的数的和大于0则加上后面的数，不然的话抛弃，直接等于后的数。

还要考虑负数，最大值用与存储。

```
 public int FindGreatestSumOfSubArray(int[] array) {
        if(array.length == 0)
            return 0;
        int cur = array[0], max = array[0];
        for(int i=1; i<array.length; i++){
            cur = cur > 0 ? cur + array[i] : array[i];
            if(max < cur)
                max = cur;
        }
        return max;
    }
```
### 整数1出现次数
求出1~13的整数中1出现的次数,并算出100~1300的整数中1出现的次数？为此他特别数了一下1~13中包含1的数字有1、10、11、12、13因此共出现6次,但是对于后面问题他就没辙了。ACMer希望你们帮帮他,并把问题更加普遍化,可以很快的求出任意非负整数区间中1出现的次数（从0  到 n 中1出现的次数）。

- 法一：依次遍历每个数，判断每个数里面是否包含1
- 法二：同法一，将数字转成字符串，直接判断
- 法三：归纳法
```
//一般都是先想出法1

public int NumberOf1Between1AndN_Solution(int n) {
    int res = 0;
    for(int i = 0; i <= n; i++)
        res += number1(i);
    return res;
}

public int number1(int n){
    int res = 0;
    while(n>0){
        if(n % 10 == 1)
            res++;
        n /= 10;
    }
    return res;
}
```

```
//法3
public int NumberOf1Between1AndN_Solution(int n) {
    int res = 0;
    int cur = 0, before = 0, after = 0;
    int i = 1;
    while(i<=n){
        before = n/(i*10);
        cur = (n/i)%10;
        after = n - n/i*i;
        if(cur == 0){
            // 如果为0,出现1的次数由高位决定,等于高位数字 * 当前位数
            res += before * i;
        }else if(cur == 1){
            // 如果为1, 出现1的次数由高位和低位决定,高位*当前位+低位+1
            res += before * i + after + 1;
        }else{
            // 如果大于1, 出现1的次数由高位决定,（高位数字+1）* 当前位数
            res += (before + 1) * i;
        }
        i *= 10;
    }
    return res;
}
```
### 数组排成最小数
输入一个正整数数组，把数组里所有数字拼接起来排成一个数，打印能拼接出的所有数字中最小的一个。例如输入数组{3，32，321}，则打印出这三个数字能排成的最小数字为321323。
```
public static String PrintMinNumber(int [] numbers) {
    if(numbers.length == 0){
        return null;
    }else if(numbers.length == 1){
        return (numbers[0]+"");
    }
    
    for (int i = 0; i < numbers.length; i++) {
        int min = i;
        for (int j = i+1; j < numbers.length; j++) {
            StringBuilder temp1 = new StringBuilder();
            StringBuilder temp2 = new StringBuilder();
            int a = Integer.parseInt(temp1.append(numbers[min]).toString()+numbers[j]);
            int b = Integer.parseInt(temp2.append(numbers[j]).toString()+numbers[min]);
            if(a>b){
                min = j;
            }
        }
        swap(numbers,min,i);
    }
    
    StringBuilder res = new StringBuilder();
    
    for (int a:
         numbers) {
        res.append(a);
    }
    return res.toString();
}
```
### 丑数（*）
把只包含质因子2、3和5的数称作丑数（Ugly Number）。例如6、8都是丑数，但14不是，因为它包含质因子7。 习惯上我们把1当做是第一个丑数。求按从小到大的顺序的第N个丑数。
```
//暴力求解
public boolean isUgly(int number){
	while(number % 2 == 0)
		number/=2;
	while(number % 3 == 0)
		number /=3;
	while(number % 5 == 0)
		number /=5;
	return (number ==1)? true:false;
}
public int getUglyNumber(int index){
	if(index <= 0)
		return 0;
	int number = 0;
	int uglyFound = 0;
	while(uglyFound < index){
		number++;
		if(isUgly(number)){
			++uglyFound;
		}
	}
	return number;
}

```
丑数一定是由某个较小的丑数乘以2/3/5得到的。因为是按递增排列的，所以选择出最小的一个。

当ugly’中只有数字1时，比较三个数的大小[1 * 2, 1 * 3, 1 * 5],选择2加入ugly中。这时需要比较大小的数字变成了5个【3,5， 4,6,10】。

因为第一次选择的是乘2的结果，所以第二次可定不会再选择1*2作为一个比较对象了。但是注意到一个数p，乘以2/3/5，最小的结果一定是乘以2的，所以只需要拿出2 * 2 = 4来和3和5比较找出最小的值即可。现在的ugly变成了【1,2,3】，下一次需要比较的数字有7个【5; 4,6,10; 6,9,15】.

同上一次比较时的想法一样，每一组中选择一个最小的进行比较，即【5,4,6】，选择4.
ugly【1,2,3,4】;【5;6,10; 6,9,15; 8,12,20】

注意到，【4,6,10】和【8,12,20】虽然是不同数字乘以2/3/5的结果（2和4），但是4是2乘以2得到的，显然这一组数的大小很容易判断的，因此仍然只需要比较三个数【5,6,9】.

观察发现，每次比较的三个数，不一定是最小的三个数，单都是分别属于已知丑数乘以2/3/5的结果。基于此，考虑用三个标记来记录。具体参见代码。
注：丑数的前6个数，分别是【1,2,3,4,5,6】,基于此，才有代码第一个index < 7 的判断。

```
//O（N）创建数组保存已经找到的丑数，用空间换时间的解法：

 public int GetUglyNumber_Solution(int index) {
        if(index <= 0)
            return 0;
        if(index == 1)
            return 1;
        int t2 = 0, t3 = 0, t5 = 0;
        int [] res = new int[index];
        res[0] = 1;
        for(int i = 1; i<index; i++){
            res[i] = Math.min(res[t2]*2, Math.min(res[t3]*3, res[t5]*5));
            if(res[i] == res[t2]*2) t2++;
            if(res[i] == res[t3]*3) t3++;
            if(res[i] == res[t5]*5) t5++;
        }
        return res[index-1];
    }

```
### 第一个只出现一个的字符

```
//利用HashMap
public static int FirstNotRepeatingChar(String str) {
    HashMap<Character,Integer> res = new HashMap<>();
    char[] strTemp = str.toCharArray();

    for (int i = 0; i < str.length(); i++) {
        if(res.containsKey(strTemp[i])){
            res.put(strTemp[i],-1);
        }else {
            res.put(strTemp[i],i);
        }
    }
    for (int i = 0; i < str.length(); i++) {
        if(res.get(strTemp[i] )!= -1){
            return i;
        }
    }

    return -1;

}
```
### 数组中的逆序对(**)
在数组中的两个数字，如果前面一个数字大于后面的数字，则这两个数字组成一个逆序对。输入一个数组,求出这个数组中的逆序对的总数P。


```
//笨办法：采用一一比较
public static int InversePairs(int [] array) {
    int res = 0;
    for (int i = 0; i < array.length; i++) {

        for (int j = i+1; j <  array.length; j++) {
            if(array[i]>array[j]){
                res++;
            }
            if(res>1000000007){
                res%=1000000007
            }
        }
    }
    return res%1000000007;
}
```



```
//采用分治的策略

//先把数组分解成两个长度为2的子数组
//再把这两个子数组分别拆分成两个长度为1的子数组
//接下来一边合并相邻的子数组，以便统计逆序对的数目。
//在第一对长度为1的子数组｛7｝、｛5｝中，7大于5，因此（7，5）也是一个逆序对。同样的，
//在第二对长度为1的子数组｛6｝、｛4｝中，也有逆序对（6，4）。
//由于已经统计了这两对子数组内部的逆序对了，
//因此需要把这两对子数组排序，以免在以后的统计过程中再避免重复。 

public int InversePairs(int [] array) {
    int len = array.length;
    if(array== null || len <= 0){
        return 0;
    }
    return mergeSort(array, 0, len-1);
}

public int mergeSort(int [] array, int start, int end){
    if(start == end)
        return 0;
    int mid = (start + end) / 2;
    int left_count = mergeSort(array, start, mid);
    int right_count = mergeSort(array, mid + 1, end);
    int i = mid, j = end;
    int [] copy = new int[end - start + 1];
    int copy_index = end - start;
    int count = 0;
    while(i >= start && j >= mid + 1){
        if(array[i] > array[j]){
            copy[copy_index--] = array[i--];
            count += j - mid;
            if(count > 1000000007){
                count %= 1000000007;
            }
        }else{
            copy[copy_index--] = array[j--];
        }
    }
    while(i >= start){
        copy[copy_index--] = array[i--];
    }
    while(j >= mid + 1){
        copy[copy_index--] = array[j--];
    }
    i = 0;
    while(start <= end) {
        array[start++] = copy[i++];
    }
    return (left_count+right_count+count)%1000000007;
}
```
### 输入两个链表，找出第一个公共结点

```
//从尾巴开始进行比较
public ListNode FindFirstCommonNode(ListNode pHead1, ListNode pHead2) {
    if(pHead1 == null || pHead2 == null)
        return null;
    int count1 = 1, count2 = 1;
    ListNode p1 = pHead1;
    ListNode p2 = pHead2;
    while(p1.next != null){
        p1 = p1.next;
        count1++;
    }
    while(p2.next != null){
        p2 = p2.next;
        count2++;
    }
    if(count1>count2){
        int dif = count1 - count2;
        while(dif != 0){
            pHead1 = pHead1.next;
            dif--;
        }
    }else{
        int dif = count2 - count1;
        while(dif != 0){
            pHead2 = pHead2.next;
            dif--;
        }
    }
    while(pHead1 != null && pHead2 != null){
        if(pHead1 == pHead2)
            return pHead1;
        pHead1 = pHead1.next;
        pHead2 = pHead2.next;
    }
    return null;
}
```

```
//利用stack
public ListNode FindFirstCommonNode(ListNode pHead1, ListNode pHead2) {
    Stack<ListNode> s1 = new Stack<>();
    Stack<ListNode> s2 = new Stack<>();
    while(pHead1 != null){
        s1.push(pHead1);
        pHead1 = pHead1.next;
    }
    while(pHead2 != null){
        s2.push(pHead2);
        pHead2 = pHead2.next;
    }

    ListNode res = null;
    while(!s1.isEmpty() && !s2.isEmpty() && s1.peek() == s2.peek()){
        s1.pop();
        res = s2.pop();
    }
    return res;
}
```
### 有序数组中某数的个数

```
//求范围，某数+0.5和某书-0.5之间的值的数量
public int GetNumberOfK(int [] array , int k) {
    return biSearch(array, k+0.5) - biSearch(array, k-0.5);
}
public int biSearch(int [] array, double k){
    int start  = 0, end = array.length - 1;
    while(start <= end){
        int mid = start + (end - start)/2;
        if(array[mid] > k){
            end = mid - 1;
        }else{
            start = mid + 1;
        }
    }
    return start;
}
```
### 二叉树的层数（*）

```
//递归
public int TreeDepth(TreeNode root) {
    if(root == null)
        return 0;
    int left = TreeDepth(root.left) + 1;
    int right = TreeDepth(root.right) + 1;
    return left > right ? left : right;
}
```

```
public static int TreeDepth(TreeNode root) {
    
    if(root == null){
        return 0;
    }
    int deep = 0;
    Queue<TreeNode> queue = new LinkedList<TreeNode>();
    queue.offer(root);
    int start = 0, end = 1;
    while(!queue.isEmpty()){
        TreeNode node = queue.poll();
        start ++;
        if(node.left != null){
            queue.offer(node.left);
        }
        if(node.right != null){
            queue.offer(node.right);
        }
        if(start == end){
            end = queue.size();
            start = 0;
            deep ++;
        }
    }
    
    return deep;

}
```
### 判断否是平衡二叉树（*）
它是一棵空树或它的左右两个子树的高度差的绝对值不超过1
```
public boolean IsBalanced_Solution(TreeNode root) {
   if(root == null){
       return true;
   } 
   if( Math.abs(getDepth(root.left) - getDepth(root.right)) <= 1 ) {
        //满足左右子树高度差小于等于1,那就接着判断左右子树是不是二叉树
        return (IsBalanced_Solution(root.left) && IsBalanced_Solution(root.right));
   } else {
        //不满足左右子树高度差小于等于1,那这棵树肯定不是平衡二叉树啦
        return false;
   }

}

public int getDepth(TreeNode root){
    if(root == null){
        return 0;
    }
    int left = getDepth(root.left)+1;
    int right = getDepth(root.right)+1;
    return left>right?left:right;
}
```
### 数组中只出现一次的数字

```
public void FindNumsAppearOnce(int [] array,int num1[] , int num2[]) {
    if(array.length == 0){
       return;
    }
   
    HashMap<Integer, Integer> temp = new HashMap<Integer, Integer>();
    for(int i = 0; i < array.length; i++){
        if(temp.containsKey(array[i]))
            temp.remove(array[i]);
        else
            temp.put(array[i], 1);
    }
    
    Iterator it = temp.entrySet().iterator();
    Map.Entry entry = (Map.Entry) it.next();
    num1[0] = entry.getKey();
    entry = (Map.Entry) it.next();
    num2[0] = entry.getKey();
 
  
}
```

```
//异或，每个相同的异或最后为0
//性能没有比之前的HashMap好很多
 public void FindNumsAppearOnce(int [] array,int num1[] , int num2[]) {
    num1[0] = 0;
    num2[0] = 0;
    if(array.length == 0)
        return;
    int num = 0;
    for(int i = 0; i < array.length; i++){
        num ^= array[i];
    }
    int index = 0;
    while((num & 1) == 0 && index < 8){
        num = num >> 1;
        index ++;
    }

    for(int i = 0; i < array.length; i++){
        if(isa1(array[i], index))
            num1[0] ^= array[i];
        else
            num2[0] ^= array[i];
    }
}

public boolean isa1(int i, int index){
    i = i >> index;
    return (i & 1) == 1;
}

```
### 和为S的连续数列

```
public ArrayList<ArrayList<Integer> > FindContinuousSequence(int sum) {
    ArrayList<ArrayList<Integer> > res = new ArrayList<ArrayList<Integer> >();
    if(sum < 3)
       return res;
    int start = 1, end = 2, mid = (1+sum)/2;//两个中间以上的值肯定大于sum
    while(start < mid){
        int s = totalSum(start, end);//求和
        if(s == sum){
            res.add(getSequence(start, end));//放入列队
            end ++;
        }else if(s < sum){
            end ++;
        }else if(s > sum){
            start ++;
        }
    }
    return res;
}

public int totalSum(int start, int end){//得到和
    int sum = 0;
    for(int i = start; i <= end; i++){
        sum += i;
    }
    return sum;
}

public ArrayList<Integer> getSequence(int start, int end){
    ArrayList<Integer> temp = new ArrayList<>();
    for(int i = start; i <= end; i++){
        temp.add(i);
    }
    return temp;
}
```
### 和为S的两个数

```
public ArrayList<Integer> FindNumbersWithSum(int [] array,int sum) {
    ArrayList<Integer> res = new ArrayList<Integer>();
    if(array.length < 2)
        return res;
    HashMap<Integer, Integer> map = new HashMap<Integer, Integer>();
    int less = Integer.MAX_VALUE;
    for(int i = 0; i < array.length; i++){
        if(map.containsKey(array[i])){//如果有对应的值
            if(array[i] * map.get(array[i]) < less){//进行比较
                less = array[i] * map.get(array[i]);
                res.clear();
                res.add(map.get(array[i]));
                res.add(array[i]);
            }
        }else{
            int key = sum - array[i];//放入HashMap
            map.put(key, array[i]);
        }
    }
    return res;
}
```
### 左旋转字符串
```
  public String LeftRotateString(String str,int n) {
        int len = str.length();
        if(len == 0)
            return "";
        n = n % len;
        String s1 = str.substring(n, len);
        String s2 = str.substring(0, n);
        return s1+s2;
    }
```
### 翻转单词顺序列
```
    public String ReverseSentence(String str) {
        if(str.trim().length() == 0)
            return str;
        String [] temp = str.split(" ");
        String res = "";
        for(int i = temp.length - 1; i >= 0; i--){
            res += temp[i];
            if(i != 0)
                res += " ";
        }
        return res;
    }
```
### 扑克牌顺子

```
public boolean isContinuous(int [] numbers) {
    int zero = 0, dis = 0;
    if(numbers.length != 5)
        return false;
    Arrays.sort(numbers);
    for(int i = 0; i < 4; i++){
        if(numbers[i] == 0){//统计王的数量
            zero ++;
            continue;
        }
        if(numbers[i] == numbers[i+1])//相同的牌就直接GG
            return false;
        if(numbers[i+1] - numbers[i] > 1){
            dis += numbers[i+1] - numbers[i] - 1;//统计顺子中间差几张
        }
    }
    if(zero >= dis)
        return true;
    else
        return false;
}
```
### 圆圈中最后剩下的数

```
//约瑟夫环
    int LastRemaining_Solution(int n, int m)
    {
        if(!n&&!m)return -1;
        int lastpeople=0;
        for (int i = 2; i <= n; i++)
        {
            lastpeople = (lastpeople + m) % i;
        }
        return lastpeople;
    }
```


```
//链表模拟
 public int LastRemaining_Solution(int n, int m) {
        if(n < 1 || m < 1)
            return -1;
        LinkedList<Integer> link = new LinkedList<Integer>();
        for(int i = 0; i < n; i++)
            link.add(i);
        int index = -1;   //起步是 -1 不是 0
        while(link.size() > 1){
            index = (index + m) % link.size();  //对 link的长度求余不是对 n
            link.remove(index);
            index --;//从这个点开始
        }
        return link.get(0);
    }
```
### 1+...+N
求1+2+3+...+n，要求不能使用乘除法、for、while、if、else、switch、case等关键字及条件判断语句（A?B:C）。
```java
//递归
 public int Sum_Solution(int n) {
        if(n==1){
            return 1;
        }
        return n+Sum_Solution(n-1);
    }
```
### 不用加减乘除做加法



1. 两数进行异或：  0011^0101=0110 这个数字其实是把原数中不需进位的二进制位进行了组合
2. 两数进行与：    0011&0101=0001 这个数字为1的位置表示需要进位，而进位动作是需要向前一位进位
3. 左移一位：      0001<<1=0010

此时我们就完成0011 + 0101 = 0110 + 0010的转换

如此转换下去，直到其中一个数字为0时，另一个数字就是原来的两个数字的和
```
3+5
3^5 = 6
3&5 = 1
1<<1 = 2
6+2 = 8
```
```java
    public int Add(int num1, int num2) {
        while (num2 != 0) {
            int temp = num1 ^ num2;
            num2 = (num1 & num2) << 1;
            num1 = temp;
        }
        return num1;
    }
```
### 把字符串转换成整数
```
 public int StrToInt(String str) {
        if(str.length() == 0)
            return 0;
        boolean flag = true;
        int start = 0;
        if(str.charAt(0) == '+')
            flag = true;
            start = 1;
        else if(str.charAt(0) == '-')
            flag = false;
            start = 1;
        
        long res = 0;
        for(int i=start;i<str.length();i++ ){
            if(str.charAt(i) > '9' || str.charAt(i) < '0')
                return 0;
            res = res * 10 + (str.charAt(i) - '0');
            start ++;
        }
        return flag == false ? -(int)res : (int)res;
    }
```
### 数组中重复的数字
```
//1.交换
  public boolean duplicate(int numbers[],int length,int [] duplication) {
        if(length == 0)
            return false;
        for(int i = 0; i < length; i++){
            while(i != numbers[i]){
                if(numbers[i] == numbers[numbers[i]]){
                    duplication[0] = numbers[i];
                    return true;
                }else{
                    int temp = numbers[numbers[i]];
                    numbers[numbers[i]] = numbers[i];
                    numbers[i] = temp;
                }
            }
        }
        return false;
    }
```

```
2. 使用HashMap进行存储，然后遍历输出
```
### 构建乘积数组

```
//推荐
 public static int[] multiply(int[] A) {
        if(A==null || A.length<2)
            return null;
        int[] B=new int[A.length];
        B[0]=1;
        for(int i=1;i<A.length;i++)
            B[i]=B[i-1]*A[i-1];
        int temp=1;
        for(int i=A.length-2;i>=0;i--){
            temp*=A[i+1];
            B[i]*=temp;
        }
        return B;
    }

```

```
//自己写的
public static int[] result(int[] A){
        int[] res = new int[A.length];
        int[] head = new int[A.length];
        int[] tail = new int[A.length];
        int temp = 1,temp1=1;
        for (int i = 0; i < A.length; i++) {
            head[i] = temp*A[i];
            temp= head[i];
        }
        for (int i = A.length-1; i >= 0; i--) {
            tail[i] = temp1*A[i];
            temp1 = tail[i];
        }
        res[0]= tail[1];
        res[res.length-1]=head[head.length-2];
        for (int i = 1; i < res.length-1; i++) {
            res[i] = head[i-1]*tail[i+1];
        }
        return res;
    }
```
### 正则匹配（**）
当模式中的第二个字符不是“*”时：
- 1、如果字符串第一个字符和模式中的第一个字符相匹配，那么字符串和模式都后移一个字符，然后匹配剩余的。
- 2、如果 字符串第一个字符和模式中的第一个字符相不匹配，直接返回false。

而当模式中的第二个字符是“*”时：
1. 如果字符串第一个字符跟模式第一个字符不匹配，则模式后移2个字符，继续匹配。如果字符串第一个字符跟模式第一个字符匹配，可以有3种匹配方式：
    - 1、模式后移2字符，相当于x*被忽略；
    - 2、字符串后移1字符，模式后移2字符；
    - 3、字符串后移1字符，模式不变，即继续匹配字符下一位，因为*可以匹配多位。

如果是“.”
- 从头到尾两个一一比较
- 如实是不匹配 false
- 如果是“.”不匹配也算 true
```
public class Solution {
     public boolean match(char[] str, char[] pattern)
    {
              if(str==null||pattern==null) return false;
        
		return matchCore(str,0,pattern,0);
    }
	
	public boolean matchCore(char[] str,int s, char[] pattern,int p) {
		if(str.length<=s&&pattern.length<=p)
			return true;//都匹配完了
		if(str.length>s&&pattern.length<=p)
			return false;//模式完了，字符串还有
		//模式串a*a没结束，匹配串可结束可不结束
		
		if(p+1<pattern.length&&pattern[p+1]=='*'){//当前pattern的下一个是*号时
			
			//字符串完了
			if(s>=str.length) return matchCore(str, s, pattern, p+2);
			else{
			
			if(pattern[p]==str[s]||pattern[p]=='.'){
				//当前位置匹配完成，移动到下一个模式串
				return matchCore(str,s+1, pattern,p+2)
						||matchCore(str,s+1, pattern,p)
						||matchCore(str,s, pattern,p+2);
			}else
				return matchCore(str, s, pattern, p+2);
			}
		}
		//当前pattern的下一个不是*时候
		if(s>=str.length) return false;
		else{
		if(str[s]==pattern[p]||pattern[p]=='.')
			return matchCore(str, s+1, pattern, p+1);
		}
		return false;
    }   
}
```
### 表示数值的字符串

设置三个标志符分别记录“+/-”、“e/E”和“.”是否出现过。

- 对于“+/-”： 正常来看它们第一次出现的话应该出现在字符串的第一个位置，如果它第一次出现在不是字符串首位，而且它的前面也不是“e/E”，那就不符合规则；如果是第二次出现，那么它就应该出现在“e/E”的后面，如果“+/-”的前面不是“e/E”，那也不符合规则。
- 对于“e/E”： 如果它的后面不接任何数字，就不符合规则；如果出现多个“e/E”也不符合规则。
- 对于“.”： 出现多个“.”是不符合规则的。还有“e/E”的字符串出现“.”也是不符合规则的。
- 同时，要保证其他字符均为 0-9 之间的数字。
```
    public boolean isNumeric(char[] str) {
        int len = str.length;
        //sign代表‘+’‘-’出现第二次，hasE代表e/E出现第二次，decimal代表'.'出现第二次
        boolean sign = false, decimal = false, hasE = false;
      
        for(int i = 0; i < len; i++){
            if(str[i] == '+' || str[i] == '-'){
                if(!sign && i > 0 && str[i-1] != 'e' && str[i-1] != 'E')
                    return false;
                if(sign && str[i-1] != 'e' && str[i-1] != 'E')
                    return false;
                sign = true;
            }else if(str[i] == 'e' || str[i] == 'E'){
                if(i == len - 1)
                    return false;
                if(hasE)
                    return false;
                hasE = true;
            }else if(str[i] == '.'){
                if(hasE || decimal)
                    return false;
                decimal = true;
            }else if(str[i] < '0' || str[i] > '9')
                return false;
        }
        return true;
    }
```

### 字符流中第一个不重复的字符

```
public class Solution {
    HashMap<Character, Integer> map = new HashMap<Character, Integer>();
    StringBuffer s = new StringBuffer();
    //插入
    public void Insert(char ch)
    {
        s.append(ch);
        if(map.containsKey(ch)){
            map.put(ch, map.get(ch)+1);
        }else{
            map.put(ch, 1);
        }
    }
    //寻找
    public char FirstAppearingOnce()
    {
        for(int i = 0; i < s.length(); i++){
            if(map.get(s.charAt(i)) == 1)
                return s.charAt(i);
        }
        return '#';
    }
}
```
### 找出链表有环处

```
  public ListNode EntryNodeOfLoop(ListNode pHead)
    {
    //快慢指针找是否有环
        ListNode fast = pHead;
        ListNode slow = pHead;

        while (fast!=null){
            if(fast.next != null&&fast.next.next!=null){
                fast = fast.next.next;
            }else {
                return null;
            }
            slow = slow.next;

            //有环了
            if(slow == fast){
                fast = pHead;
                //一个指针从头，一个指针从判断处 slow==fast，遇到就是环处
                while(fast!=slow){
                    fast = fast.next;
                    slow = slow.next;
                }
                return fast;
            }
        }
        return null;
    }
```
### 删除链表中重复的结点
```
 public ListNode deleteDeplication(ListNode pHead){
 
        if (pHead == null)
            return null;
 
        //注意备用头结点，头结点可能被删除
        ListNode first = new ListNode(-1);
 
        first.next = pHead;
        ListNode p = pHead;
        //前节点
        ListNode preNode = first;
 
        while (p != null && p.next != null){
 
            if (p.val == p.next.val){ //两节点相等
 
                int val = p.val; //记录val用于判断后面节点是否重复
                while(p != null && p.val == val){   //这一步用于跳过相等的节点，用于删除
                    p = p.next;
                }
                preNode.next = p; //删除操作，前节点的next直接等于现在的节点，把中间的节点直接跨过
            }else {
                preNode = p;
                p = p.next;
            }
        }
        return first.next;
    }

```
### 二叉树的下一个结点
```
public TreeLinkNode GetNext(TreeLinkNode pNode)
    {
        if(pNode == null){//为空
            return null;
        }
        if(pNode.right != null){ //存在右子树
            TreeLinkNode node = pNode.right;
            while(node.left != null){
                node = node.left;
            }
            return node;
        }
        while(pNode.next != null){ //不存在右子树，看有没有父节点
            TreeLinkNode root = pNode.next;
            if(pNode == root.left)
                return root;
            pNode = root;
        }
        return null;
    }s
```
### 判断二叉树是否对称

```
    boolean isSymmetrical(TreeNode pRoot)
    {
        if(pRoot == null)
            return true;
        return test(pRoot.left, pRoot.right);
    }
    boolean test(TreeNode left, TreeNode right){
        if(left == null && right == null)
            return true;
        if(left == null || right == null)
            return false;
        if(left.val == right.val){
            return test(left.left, right.right) && 
                test(left.right, right.left);
        }
        return false;
    }
```



### 按之字形顺序打印二叉树
```

 public ArrayList<ArrayList<Integer> > Print(TreeNode pRoot) {
        ArrayList<ArrayList<Integer>> res = new ArrayList<>();
        if(pRoot == null){
            return res;
        }
        Stack<TreeNode> one = new Stack<>();
        Stack<TreeNode> two = new Stack<>();
        one.push(pRoot);
        ArrayList<Integer> temp = new ArrayList<>();
        while(!one.empty()|| !two.empty()){

            if(!one.empty()){
               while(!one.empty()){ 
                   TreeNode t= one.pop();
                   if(t.left!=null){
                       two.push(t.left);
                   }
                   if(t.right!=null){
                       two.push(t.right);
                   }

                   temp.add(t.val);
               }
                res.add(new ArrayList<Integer>(temp));
                temp.clear();
            }
            if(!two.empty()){
                while(!two.empty()){//因为栈，所以要反着来
                    TreeNode t= two.pop();
                    if(t.right!=null){
                        one.push(t.right);
                    }
                    if(t.left!=null){
                        one.push(t.left);
                    }
                    temp.add(t.val);
                }
                res.add(new ArrayList<Integer>(temp));
                temp.clear();
            }
        }
        return res;

    }
```
### 把二叉树打印成多行
```
   ArrayList<ArrayList<Integer> > Print(TreeNode pRoot) {
        ArrayList<ArrayList<Integer> > res = new ArrayList<ArrayList<Integer> >();
        if(pRoot == null)
            return res;
        ArrayList<Integer> temp = new ArrayList<Integer>();
        Queue<TreeNode> layer = new LinkedList<TreeNode>();
        layer.offer(pRoot);
        int start = 0, end = 1;
        while(!layer.isEmpty()){
            TreeNode node = layer.poll();
            temp.add(node.val);
            start ++;
            if(node.left != null)
                layer.add(node.left);
            if(node.right != null)
                layer.add(node.right);
            if(start == end){
                start = 0;
                res.add(temp);
                temp = new ArrayList<Integer>();
                end = layer.size();
            }
        }
        return res;
    }
```
### 序列化二叉树
```
import java.util.Queue;
import java.util.LinkedList;
public class Solution {
    String Serialize(TreeNode root) {
        if(root == null){
            return "#,";
        }
        StringBuffer res = new StringBuffer(root.val + ",");
        res.append(Serialize(root.left));
        res.append(Serialize(root.right));
        return res.toString();
    }
    TreeNode Deserialize(String str) {
        String [] res = str.split(",");
        Queue<String> queue = new LinkedList<String>();
        for(int i = 0; i < res.length; i++){
            queue.offer(res[i]);
        }
        return pre(queue);
    }
    TreeNode pre(Queue<String> queue){
        String val = queue.poll();
        if(val.equals("#"))
            return null;
        TreeNode node = new TreeNode(Integer.parseInt(val));
        node.left = pre(queue);
        node.right = pre(queue);
        return node;
    }
}

```
### 获取低K大的节点 二叉树

```
//中序遍历
public class Solution {
    int index = 0;
    TreeNode KthNode(TreeNode pRoot, int k)
    {
        if(pRoot != null){
            TreeNode node = KthNode(pRoot.left, k);
            if(node != null)
                return node;
            index ++;
            if(index == k)
                return pRoot;
            node = KthNode(pRoot.right, k);
            if(node != null)
                return node;
        }
        return null;
    }
```
### 中位数

```

    private static List<Double> res = new ArrayList<>();
    public void Insert(Integer num) {
        res.add((double)num);
    }

    public Double GetMedian() {
        if(res.size() == 1){
            return res.get(0);
        }
        Collections.sort(res);
        
        if(res.size()%2 == 0){
            double num = (res.get(res.size()/2-1)+res.get(res.size()/2))/2;
            return num;
        }else{
            double num = res.get(res.size()/2);
            return num;
        }
    }
```

```
  // 最小堆（右）
    private PriorityQueue<Integer> rHeap = new PriorityQueue<>(); 
    // 最大堆（左）
    private PriorityQueue<Integer> lHeap = new PriorityQueue<Integer>(15, new Comparator<Integer>() {
            public int compare(Integer o1, Integer o2) {
                return o2 - o1;
            }
        });
    // 保证lHeap.size()>=rHeap.size()
    public void Insert(Integer num) {
        // 先按大小插入，再调整
        if(lHeap.isEmpty() || num <= lHeap.peek())
            lHeap.offer(num);
        else
            rHeap.offer(num);

        if(lHeap.size() < rHeap.size()){
            lHeap.offer(rHeap.peek());
            rHeap.poll();
        }else if(lHeap.size() - rHeap.size() == 2){
            rHeap.offer(lHeap.peek());
            lHeap.poll();
        }
    }
    public Double GetMedian() {
        if(lHeap.size() > rHeap.size())
            return new Double(lHeap.peek());
        else
            return new Double(lHeap.peek() + rHeap.peek())/2;
    }
```
### 滑动窗口

```
 public ArrayList<Integer> maxInWindows(int [] num, int size){
        ArrayList<Integer> res = new ArrayList<Integer>();
        LinkedList<Integer> deque = new LinkedList<Integer>();
        if(num.length == 0 || size == 0)
            return res;
        for(int i = 0; i < num.length; i++){
            if(!deque.isEmpty() && deque.peekFirst() <= i - size)
                deque.poll();
            while(!deque.isEmpty() && num[deque.peekLast()] < num[i])
                deque.removeLast();
            deque.offerLast(i);
            if(i + 1 >= size)
                res.add(num[deque.peekFirst()]);
        }
        return res;
    }
```

```
   public ArrayList<Integer> maxInWindows(int [] num, int size)
    {
        ArrayList<Integer> res = new ArrayList<>();
        if(size>num.length||size<1){
            return res;
        }
        
        int end = size-1;
        int start  = 0;

        while(end<=num.length-1){
            int max = num[start];
            for (int i = 1; i < size; i++) {
                if(num[start+i]>max){
                    max = num[start+i];
                }
            }
            res.add(max);
            end++;
            start++;
        }
        return res;
    }
```


### 矩阵路径（**）
回溯法：
1. 将matrix字符串映射为一个字符矩阵（index = i * cols + j）
2. 遍历matrix的每个坐标，与str的首个字符对比，如果相同，用flag做标记，matrix的坐标分别上、下、左、右、移动（判断是否出界或者之前已经走过[flag的坐标为1]），再和str的下一个坐标相比，直到str全部对比完，即找到路径，否则找不到。
```
  public boolean hasPath(char[] matrix, int rows, int cols, char[] str)
    {
        if(matrix.length == 0 || str.length == 0)
            return false;
        int [][] flag = new int[rows][cols];
        for(int i = 0; i < rows; i++){
            for(int j = 0; j < cols; j++){
                if(search(matrix, rows, cols, i, j, str, 0, flag))
                    return true;
            }
        }
        return false;
    }

    public boolean search(char[] matrix, int rows, int cols, 
                          int i, int j, char[] str, int index, int[][] flag){
        int m_i = i * cols + j;
        if(i<0 || j<0 || i >= rows || j>=cols || flag[i][j] == 1 || matrix[m_i] != str[index])
            return false;
        if(index >= str.length - 1)
            return true;
        flag[i][j] = 1;
        if(search(matrix, rows, cols, i+1, j, str, index+1, flag) || 
            search(matrix, rows, cols, i-1, j, str, index+1, flag) || 
            search(matrix, rows, cols, i, j+1, str, index+1, flag) || 
            search(matrix, rows, cols, i, j-1, str, index+1, flag))
            return true;
        flag[i][j] = 0;
        return false;
    }
```
### 机器人走路（**）
回溯法：从(0,0)开始走，每成功走一步用一个flag数组标记当前位置为1，然后从当前位置往四个方向探索，
返回1 + 4 个方向的探索值之和。
```
 public int movingCount(int threshold, int rows, int cols)
    {   
        if(threshold == 0)
            return 0;
        int flag[][] = new int[rows][cols];
        return count(threshold, rows, cols, 0, 0, flag);
    }
    public int count(int threshold, int rows, int cols, int i, int j, int[][] flag){
        if(i<0 || j<0 || i>=rows || j>=cols || sum(i)+sum(j) > threshold || flag[i][j] == 1){
            return 0;
        }
        flag[i][j] = 1;
        return 1 + count(threshold, rows, cols, i - 1, j, flag) + 
            count(threshold, rows, cols, i + 1, j, flag) +
            count(threshold, rows, cols, i, j - 1, flag) +
            count(threshold, rows, cols, i, j + 1, flag);
    }
    public int sum(int i){
        int s = 0;
        while(i>0){
            s += i%10;
            i /= 10;
        }
        return s;
    }
```
