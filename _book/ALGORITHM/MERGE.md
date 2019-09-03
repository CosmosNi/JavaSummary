# 归并排序

```java
/**
 * 归并排序：
 * 将一个数组拆分成两半，分别对每一半进行排序，然后使用合并(merge)操作，把两个有序的子数组合并成一个整体的有序数组。
 * 我们可以把一个数组刚开始先分成两，也就是2个1/2，之后再将每一半分成两半，也就是4个1/4，
 * 以此类推，反复的分隔数组，直到得到的子数组中只包含一个数据项，这就是基值条件，只有一个数据项的子数组肯定是有序的。
 */
public class MergeSort {


    public static void mergeSort(int arr[], int left, int right) {
        if (left < right) {
            int mid = (right + left) / 2;
            //递归分解左边数组
            mergeSort(arr, left, mid);
            //递归分解右边数组
            mergeSort(arr, mid + 1, right);
            //分解到左右各只有一个元素时，进行合并
            merge(arr, left, right);
        }
    }

    private static void merge(int arr[], int startIndex, int endIndex) {
        //记录数组中间位置
        int mid = (endIndex + startIndex) / 2;
        //创建一个临时数组
        int[] temp = new int[endIndex - startIndex + 1];
        //左侧数组起始位置
        int leftStartIndex = startIndex;
        //右侧数组起始位置
        int rightStartIndex = mid + 1;
        //记录临时数组索引位置
        int count = 0;
        //对两个数组进行归并
        while (leftStartIndex <= mid && rightStartIndex <= endIndex) {
            if (arr[leftStartIndex] <= arr[rightStartIndex]) {
                temp[count++] = arr[leftStartIndex++];
            } else {
                temp[count++] = arr[rightStartIndex++];
            }
        }
        while (leftStartIndex <= mid) {
            temp[count++] = arr[leftStartIndex++];
        }
        while (rightStartIndex <= endIndex) {
            temp[count++] = arr[rightStartIndex++];
        }
        //复制临时数组到原数组中
        System.arraycopy(temp, 0, arr, startIndex, count);
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
        mergeSort(data, 0, data.length - 1);
        System.out.print("归并排序后：");
        printArray(data);
    }
}

```

