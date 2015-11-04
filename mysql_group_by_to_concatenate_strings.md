# MySQL Group By to Concatenate Strings

![image](http://i.imgur.com/Oo1GLWz.png)

이런 상황이 있었다. bookmark가 n개의 tag를 갖는데 이게 n개의 row가 아니라 1개의 row로 표현하고픈...

[how-to-use-group-by-to-concatenate-strings-in-mysql](http://stackoverflow.com/questions/149772/how-to-use-group-by-to-concatenate-strings-in-mysql)

아는 후배에게 물어보니 위 방법을 알려주었다.

![image](http://i.imgur.com/jySfEF8.png)

이렇게 하면 3개의 row로 표현되던 것을 1개의 row로 표현이 가능하다.

하지만 1개의 필드만 사용할 수 있다는 단점이 있다.

그래도 아주 훌룡항 방법이라고 생각한다.

그런데 다른 RDB는 array 타입도 지원하는 듯