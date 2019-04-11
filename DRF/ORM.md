### 1. related_name
model의 Field에 related_name을 옵션으로 주면 역방향으로 참조가 가능하다
```python
class User(models.Model):
    name = models.CharField(max_length=100)

class Profile(models.Model):
    owner = models.ForeignKey(User, related_name='profiles',on_delete=models.CASCADE)
#shell
my_user = User.objects.get(pk=1).profiles
```
위의 ```shell```처럼 명령을 내리면 ForeignKey로 1을 가지는 모든 profiles을 뽑아준다.


### 2. ORM에서의 AND OR연산
2.1. Q객체를 사용하기
```python
user.objects.filter(Q(username__startswith='y')|Q(username__startswith='p'))
```
2.2. queryset|queryset
```python
user.objects.filter(userid__startswith='y') | user.objects.filter(userid__startswith='p')
```

### 3. JOIN

###### 3.1 select_related와 prefetch_related
select_related는 foreignkey, one-to-one에서 사용가능  
prefetch_related는 many-to-many, many-to-one 등 모든 relationships에서 사용가능

###### 3.2 LEFT JOIN(LEFT OUTER JOIN)
values를 통해 ForeignKey모델의 정보를 가져올때 자동으로 LEFT JOIN을 실행하는 것을 확인 할 수 있었다.
```python
    Announce.objects.values('writer_id', 'writer__userid')
```
```sql
    SELECT announce.writer_id, user.userid FROM LEFT OUTER JOIN user
    ON announce.writer_id = user.id
```

### 4. ORM에서의 Subquery
Subquery를 사용하기 위해서는 django.db.models import Subquery를 사용한다.

##### 4.1 SELECT절에서의 Subquery
아래 SELECT절에 Subquery가 사용된 SQL문을 django ORM으로 작성하고싶다면 annotate와 Subquery를 사용하면 된다.
```sql
SELECT user.*, 
(SELECT announce.pay FROM announce WHERE announce.writer_id = user.id ORDER BY announce.pay DESC LIMIT 1)
FROM user
```

```python
sub = Announce.objects.filter(writer_id = OuterRef('pk')).order_by('-pay')
user = User.objects.all().annotate(highest=Subquery(sub.values('pay')[:1]))
```

##### 4.2 WHERE절에서의 Subquery
WHERE절에 Subquery를 사용하고 싶다면 Filter와 Subquery를 사용하면 된다.
```python
q1 = Announce.objects.filter(
        writer_id = Subquery(
            user.objects.filter(userid='youngwon1121').values('id')
        )
    )
```
근데 위의 예제는 아래와 같이 그냥 filter만으로도 해결가능하다
```python
q2 = Announce.objects.filter(writer__userid = 'youngwon1121')
```
SQL문을 확인해보면 q1은 WHERE절에 subquery를 사용하고, q2는 INNER JOIN을 실행한 후 WHERE을 동해 userid조건을 거는 것을 볼 수 있었다.