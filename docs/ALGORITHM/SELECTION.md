# 选择排序

```java
/**
 * 选择排序，它的工作原理是每一次从待排序的数据元素中选出最小（或最大）的一个元素，存放在序列的起始位置，直到全部待排序的数据元素排完。
 * 与冒泡排序不同的是，选择排序不会一遇到两个数字的顺序不对时，立马交换其位置，而是记录其下标，等该轮结束后进行交换
 */
public class SelectionSort {
    public static void sort(int arr[], boolean asc) {
        //当比较到arr.length - 1，整个排序结束
        for (int i = 0; i < arr.length - 1; i++) {
            int index = i;
            for (int j = i + 1; j < arr.length; j++) {
                if (asc) {
                    //升序
                    if (arr[index] > arr[j]) {
                        index = j;
                    }
                } else {
                    //降序
                    if (arr[index] < arr[j]) {
                        index = j;
                    }
                }
            }
            exchange(arr, i, index);
        }
    }

    private static void exchange(int arr[], int i, int j) {
        int temp = arr[i];
        arr[i] = arr[j];
        arr[j] = temp;
    }

    public static void main(String[] args) {
        int[] array = new int[]{1, 5, 6, 8, 9, 4, 3, 3, 3, 4, 5, 6};
        System.out.println("排序前：" + Arrays.toString(array));
        sort(array, true);
        System.out.println("升序后：" + Arrays.toString(array));
        sort(array, false);
        System.out.println("降序后：" + Arrays.toString(array));
    }
}

```

