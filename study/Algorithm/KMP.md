# KMP Algorithm

> 문자열 검색 알고리즘
> 봐도봐도 헷갈린다.

- 핵심 개념은 찾는 문자열이 "aecaed" 일 때 만약 원문에서 "aecae"까지는 있었는데 d가 안오고 다른 문자가 왔을 경우 "aecae"의 마지막 ae는 이미 나왔다고 우리에게 보증을 서줬다.
- 이를 활용하면 문자열 탐색 시간을 획기적으로 줄일 수 있다.
- 이를 위해 핵심적인 기능이 getPI라는 함수다(아직 왜 이런 이름이 통용되는지는 잘 모르겠다.)

<img src ="https://1.bp.blogspot.com/-DXQWkd1aWs0/VxkIs0ttoNI/AAAAAAAAAIk/IomLOD9riX4rVea5UWcbvn75s4FjUicSgCLcB/s1600/Capture1.PNG" />
[ref](https://bbc1246.blogspot.com/2016/08/kmp.html)

```Java
public int[] getPI(char[] keyword){
    int compareIndex = 0;
    int length = keyword.length;
    int[] pi = new int[length];

    for(int lastIndex = 1; lastIndex < length; lastIndex++){
        while(compareIndex > 0 && keyword[lastIndex] != keyword[compareIndex]){
            compareIndex = pi[compareIndex-1];
        }
        if(keyword[lastIndex] == keyword[compareIndex]){
            pi[lastIndex] = ++j;
        }
    }

    return pi;
};
```

- preffix(접두사)와 suffix(접미사)가 얼마나 많이 겹치는지 구하는 알고리즘이다.
- 이를 통해 문자열을 비교하며 비교하는 문자열이 달라졌을 때 얼마나 많이 스킵할 수 있는지 알 수 있다.
- 초깃값으로 모두 0이 선언되어 있고
- lastIndex(0~keyword.length-1)까지의 substring에 대해 preffix와 suffix가 겹칠 수 있는 최대 길이의 배열(말이 너무 어렵다)이 반환된다.
- 이건 설명보다 반복을 통해 마음으로 이해하는 편이 훨씬 쉬울 것 같다.
- while문은 마지막 인덱스와 비교하고자하는 인덱스의 값이 다를 경우

> ex
> "abababa"에 대해 lastIndex = 5일 경우 "ababa"가 기준이다.
> |length|prefix | suffix |
> |:----:|:-----:|:-------|
> |  1   |a      |a       |
> |  2   |ab     |ba      |
> |  3   |aba    |aba     |
> |  4   |abab   |baba    |
> 길이가 3인 prefix와 suffix가 서로 같고 가장 길다

- 이 예를 이용하면 만약 ababa까지 같았지만 그 이후가 다를때 aba까지만 검색한 것으로 바꿔줄 수 있다!

	
```Java
public ArrayList<Integer> kmp(char[] context, char[] keyword){
    int[] pi = getPI(keyword);
    ArrayList<Integer> indexes = new ArrayList<>();
    
    int keywordIndex = 0;
    int keywordLength = keyword.length;
    int contextLength = context.length;

    for(int contextIndex = 0; contextIndex < contextLength; contextIndex++){
	    while(keywordIndex > 0 && 
	    	  context[contextIndex] != keyword[keywordIndex]){
	            keywordIndex = pi[keywordIndex-1];
	    }
	    if(context[contextIndex] == keyword[keywordIndex]){
	        if(keywordIndex == keywordLength-1){
	            indexes.add(contextIndex - keywordIndex + 1);
	            keywordIndex = pi[keywordIndex];
	        }else{
	            keywordIndex++;
	        }
	    }
	}

	return indexes;
};
```

- 원래 이런건 보통 두고 보면 훨씬 더 쉬워졌던 것 같다
- 아마 다음에 보면 확 더 와 닿을 것이라고 생각한다.