# 希尔排序

```java
/**
 * 希尔排序：
 * 将数的个数设为n，取奇数k=n/2，将下标差值为k的书分为一组，构成有序序列。
 * <p>
 * 再取k=k/2 ，将下标差值为k的书分为一组，构成有序序列。
 * <p>
 * 重复第二步，直到k=1执行简单插入排序。
 */
public class SheelSort {

    public static void sheelSort(int[] a) {
        int d = a.length;
        while (d != 0) {
            d = d / 2;

            for (int x = 0; x < d; x++) {//分的组数
                for (int i = x + d; i < a.length; i += d) {//组中的元素，从第二个数开始
                    int j = i - d;//j为有序序列最后一位的位数
                    int temp = a[i];//要插入的元素
                    for (; j >= 0 && temp < a[j]; j -= d) {//从后往前遍历,满足交换条件才会执行
                        a[j + d] = a[j];//向后移动d位

                    }
                    //根据交换次数来决定temp放到什么位置
                    a[j + d] = temp;
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
        //int[] data = {543, 23, 45, 65, 76, 1, 456, 7, 77, 88, 3, 9};
        int[] data = {7, 6, 4, 8, 9};
        System.out.print("数组排序前：");
        printArray(data);
        System.out.print("\n");
        sheelSort(data);
        System.out.print("排序后：");
        printArray(data);
    }
}

```

