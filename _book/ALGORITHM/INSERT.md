# 插入排序

```java
/**
 * 插入排序：
 * 插入排序(Insert Sort)将待排序的数组分为2部分：有序区，无序区。其核心思想是每次取出无序区中的第一个元素，
 * 插入到有序区中。 有序与无序区划分，就是通过一个变量标记当前数组中，前多少个元素已经是局部有序了。
 */
public class InsertSort {

    public static void sort(int arr[], boolean asc) {
        //设定有序区的起始位置
        int orderlyIndex = 0;
        for (int i = orderlyIndex + 1; i < arr.length; i++) {
            int temp = arr[i]; //记录无序区中的第一个元素值
            int insertIndex = i; //初始设置位置为自己的位置
            for (int j = orderlyIndex; j >= 0; j--) {
                if (asc) {
                    //升序，所有比temp大的值都右移
                    if (temp < arr[j]) {
                        //右移
                        arr[j + 1] = arr[j];
                        //每移动一次，插入的索引减1
                        insertIndex--;
                    } else {
                        //有序区当前位置元素<=无序区第一个元素，那么之前的元素都会<=，不需要继续比较
                        break;
                    }
                } else {
                    if (temp > arr[j]) {
                        arr[j + 1] = arr[j];

                        insertIndex--;
                    } else {
                        break;
                    }
                }
            }
            orderlyIndex++;
            arr[insertIndex] = temp;
        }
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

