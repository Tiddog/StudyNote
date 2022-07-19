# 字符串

## 1. 字符串的全排列

[原题链接: LeetCode 46.全排列](https://leetcode.cn/problems/permutations/ )

[原题链接: LeetCode 47.全排列II](https://leetcode.cn/problems/permutations-ii/)

> **解题思路 :** 
> + 穷举法
> + 字典序排列

### 解题步骤(字典序做法)

+ 首先获取字符串
  + 将字符串全部转为小写或者大写(有其他要求可不转)
  + 然后将字符串转换成字符型数组方便操作
  + 再进行初次排序, 并放入结果集中
+ 初始化指针flag, 对数组进行遍历, 顺序为倒序
  + 先找到某一个字符a, 该字符符合的特征为: 右边的字符比自己的字典序大, 并且自己是从右往左数第一个符合这个条件的字符
  + 然后再从该字符的右边, 寻找首个比这个字符大的字符b
  + 交换两者位置
  + 交换后的左半部分不动(b的左半部分,包括b), b的右半部分进行升序排序
  + 得到一个结果, 存入结果集中
  + 重复上述操作, 直到指针flag指到最左边为止
+ 最后得到的List必定是以字符集从大到小排列的结果

```java
/**
 * OrderByDic
 *
 * @Author: Jane_YH
 * @Version: 1.0
 * @Since: 2022/7/18 15:51
 * @Descript: 全排列(字典序做法)
 */
public class OrderByDic {

    private static List<List<String>> res = new ArrayList<>();

    public static void main(String[] args) {
        Scanner sc = new Scanner(new InputStreamReader(System.in));
        String original = sc.nextLine().toUpperCase();
        char[] chars = original.toCharArray();
        Arrays.sort(chars);
        res.add(new ArrayList<>(Arrays.asList(String.valueOf(chars))));
        int endFlag = chars.length-1;
        while (endFlag>0) {
            if(chars[endFlag-1]<chars[endFlag]) {
                char[] left = Arrays.copyOfRange(chars,0,endFlag);
                char[] right = Arrays.copyOfRange(chars,endFlag,chars.length);
                chars = handStr(left,right,endFlag-1);
                endFlag = chars.length-1;
            }else {
                endFlag--;
            }
        }
        System.out.println(res);
    }



    public static char[] handStr(char[] left, char[] right, int index){
        int flag = right.length-1;
        while(right[flag]<=left[index]) {
            flag--;
        }
        char temp = left[index];
        left[index] = right[flag];
        right[flag] = temp;
        Arrays.sort(right);

        List<String> list = new ArrayList<String>();

        res.add(list);
        CharBuffer buffer = CharBuffer.allocate(left.length+right.length);
        buffer.put(left).put(right);
        list.add(String.valueOf(buffer.array()));
        return buffer.array();
    }


}
```
