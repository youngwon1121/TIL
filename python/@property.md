python @property의 사용법
=========================
DRF serializer코드를 보던중에 serializer.data 이런식으로 serializer을 통해 얻은 인스턴스에서 python dictionary를 얻는 내부 코드가 궁금해졌다.

github DRF/serializers.py를 열심히 찾는데 Serializer클래스에 data라는 멤버변수가 아무리 찾아도 없다. 뭔가 잘못된걸 깨닳고 삽질하던중에 @property 데코레이터가 달린 data라는 함수를 발견했다. @property에 대해 공부해 보자.

@property는 private attirbute에 대한 접근함수를 일관성있게 만들 수 있게 해주는 것이다. 

아래 코드는 color에 대한 getter와 setter을 정의한것이다.
```python
class MyRoom:
    def __init__(self):
        self._color = "red"

    @property
    def color(self):
        return self._color

    @color.setter
    def color(self, color):
        self._color = color
```
위와같이 @property와 @*.setter을 이용해 정의한 코드는 아래와 같이 사용할 수 있다.

```python
room = MyRoom()
print(room.color) #red
room.color = "blue"
print(room.color) #blue
```