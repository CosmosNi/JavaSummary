# 冒泡排序

```java
/**冒泡排序：
 * 第一轮：将数组下标为0的数值与 [1,n-1]数值进行比较，遇到比下标为0的数值大的则进行交换，第一轮结束后，最大值就防止到第一位
 * 同理，进行下一轮比较
 */
public class BubbleSort {

    public static void sort(int arr[], boolean asc) {
        //当比较到arr.length - 1，整个排序结束
        for (int i = 0; i < arr.length - 1; i++) {
            for (int j = i + 1; j < arr.length; j++) {
                if (asc) {
                    //升序
                    if (arr[i] > arr[j]) {
                        exchange(arr, i, j);
                    }
                } else {
                    //降序
                    if (arr[i] < arr[j]) {
                        exchange(arr, i, j);
                    }
                }
            }
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

