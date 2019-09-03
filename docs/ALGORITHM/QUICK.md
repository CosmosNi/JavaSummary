# 快速排序

```java
/**
 * 快速排序（6  1  2 7  9  3  4  5 10  8）
 * 1.选择基准值，比如选第一个6为基准值
 * 2.从i=1 开始向右寻找第一个比6大的数，j从最后一个值往前找，找到第一个比6小的数，交换i,j
 * 3.i,j继续往前走，继续做类似第二步的交换
 * 4.知道i和j相遇了，或者i>j了，则停止，这时候基准值左边都比基准值小，右边都比基准值大
 * 5.此时基准值位于j所处位置，将已j分成两个数组，继续进行排序
 */
public class QuickSort {
    public static void sort(int array[], int leftIndex, int rightIndex) {
        if (leftIndex >= rightIndex) {
            return;
        }
        int partitionIndex = doSort(array, leftIndex, rightIndex);

        sort(array, leftIndex, partitionIndex - 1);
        sort(array, partitionIndex + 1, rightIndex);
    }


    public static int doSort(int[] arr, int start, int end) {
        int pivot = getPivot(arr, start, end);
        int left_pointer = start - 1;
        int right_pointer = end + 1;
        while (true) {
            //left_pointer当遇到比基准值大的元素，停下来
            while (arr[++left_pointer] < pivot) {
            }
            //right_pointer当遇到比基准值小的元素，停下来
            while (arr[--right_pointer] > pivot) {
            }
            if (left_pointer >= right_pointer) {
                break;
            }
            int temp = arr[left_pointer];
            arr[left_pointer] = arr[right_pointer];
            arr[right_pointer] = temp;
        }
        return right_pointer;
    }

    /**
     * 获取基准值
     */
    private static int getPivot(int array[], int start, int end) {
        return array[start];
    }

    public static void main(String[] args) {
        // int[] arr = {3,4,2,0,4,7,9,6,5,8};
        int[] arr = {3, 3, 3, 3, 4, 7, 2, 6, 5, 8};
        System.out.println("排序前数组:" + Arrays.toString(arr));
        sort(arr, 0, arr.length - 1);
        System.out.println("排序前数后:" + Arrays.toString(arr));
        Arrays.sort(arr);
    }
}

```

