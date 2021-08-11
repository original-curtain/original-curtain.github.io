---
title: algorithm-回溯算法
date: 2021-08-11 10:21:51
tags: algorithm
---
回溯算法实际上是一个类似枚举的搜索尝试算法，主要在搜索尝试过程中寻找问题的解，当发现不满足要求时，就回溯返回，尝试别的路径。其经典解决的问题有八皇后问题、01背包问题等。
<!--more-->
以下以leetcode的[17. 电话号码的字母组合](https://leetcode-cn.com/problems/letter-combinations-of-a-phone-number/)为例来解释这个算法。

# 题目描述
给定一个仅包含数字 2-9 的字符串，返回所有它能表示的字母组合。答案可以按 任意顺序 返回。

给出数字到字母的映射如下（与电话按键相同）。注意 1 不对应任何字母。

# 代码详解
```
public class S0017 {
    public List<String> letterCombinations(String digits) {
        List<String> ans=new ArrayList<>();
        if(digits.length()==0){
            return ans;
        }
        Map<Character,String> map=new HashMap<>();
        map.put('2',"abc");
        map.put('3',"def");
        map.put('4',"ghi");
        map.put('5',"jkl");
        map.put('6',"mno");
        map.put('7',"pqrs");
        map.put('8',"tuv");
        map.put('9',"wxyz");

        backTrack(ans,map,digits,0,new StringBuffer());
        return ans;
    }

    private void backTrack(List<String> ans,Map<Character,String> map,String digits,int index,StringBuffer combination){
        if(index==digits.length()){
            ans.add(combination.toString());
        }else{
            char digit=digits.charAt(index);
            String letters=map.get(digit);
            int letterCount=letters.length();
            for(int i=0;i<letterCount;i++){
                combination.append(letters.charAt(i));
                backTrack(ans,map,digits,index+1,combination);
                combination.deleteCharAt(index);
            }
        }
    }

    public static void main(String[] args) {
        S0017 s0017=new S0017();
        List<String> ans=s0017.letterCombinations("23");
        System.out.println(Utils.printList(ans));
    }
}
```
其中combination.deleteCharAt(index);就是删除已经过的路径从而进行回溯