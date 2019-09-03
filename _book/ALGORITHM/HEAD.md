# 堆排序

![堆排序](../image/%E5%A0%86%E6%8E%92%E5%BA%8F.png)

```java
/**
 * 堆排序
 * 1.构建大根堆（小根堆），找到第一个非叶子节点，从左至右，从下至上进行调整。将最大值放置到父节点处
 * 2.继续对第二个，第三个非叶子节点进行处理，直到到达栈顶，此时的栈顶即为最大值
 * 3.将栈顶元素和最末尾的数进行交换，继续下一次的建堆
 */
public class HeapSort {

    public static void heapSort(int[] a) {
        System.out.println("开始排序");
        int arrayLength = a.length;
        //循环建堆
        for (int i = 0; i < arrayLength - 1; i++) {
            //建堆，每次建堆都会将最大值置为栈顶
            buildMaxHeap(a, arrayLength - 1 - i);
            //交换堆顶和最后一个元素
            swap(a, 0, arrayLength - 1 - i);
            //System.out.println(Arrays.toString(a));
        }
    }

    private static void swap(int[] data, int i, int j) {
        int tmp = data[i];
        data[i] = data[j];
        data[j] = tmp;
    }

    //叶子节点是指没有子节点的节点
    //对data数组从0到lastIndex建大顶堆
    private static void buildMaxHeap(int[] data, int lastIndex) {
        //从lastIndex处节点（最后一个节点）的父节点开始（此节点的父节点为第一个非叶子节点）
        //从该节点往上，每个节点都为非叶子节点
        for (int i = (lastIndex - 1) / 2; i >= 0; i--) {
            //k保存正在判断的节点
            int k = i;
            //如果当前k节点的子节点存在，左节点等于k*2+1,右节点等于K*2+2
            while (k * 2 + 1 <= lastIndex) {
                //k节点的左子节点的索引
                int biggerIndex = 2 * k + 1;
                //如果biggerIndex小于lastIndex，即biggerIndex+1代表的k节点的右子节点存在
                if (biggerIndex < lastIndex) {
                    //若果右子节点的值较大
                    if (data[biggerIndex] < data[biggerIndex + 1]) {
                        //biggerIndex总是记录较大子节点的索引
                        biggerIndex++;
                    }
                }
                //如果k节点的值小于其较大的子节点的值
                if (data[k] < data[biggerIndex]) {
                    //交换他们
                    swap(data, k, biggerIndex);
                    //将biggerIndex赋予k，
                    //开始while循环的下一次循环，重新保证k节点的值大于其左右子节点的值
                    k = biggerIndex;
                } else {
                    break;
                }
            }
        }
    }

    public static void printArray(int arr[]) {
        for (int k = 0; k < arr.length; k++) {
            System.out.print(arr[k] + "\t");
        }
    }

    public static void main(String[] args) {
        int[] data = {543, 23, 45, 65, 76, 1, 456, 7, 77, 88, 3, 9};
        System.out.print("数组排序前：");
        printArray(data);
        System.out.print("\n");
        heapSort(data);
        System.out.print("排序后：");
        printArray(data);
    }

}

```

