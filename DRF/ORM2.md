DJANGO ORM2
========================

아래의 모델이 사용되어진다.
```python
class User(models.Model):
    userid = models.CharField()

class Announce(models.Model):
    writer = models.ForeignKey(User)
    pay = models.IntegerField()
    title = models.CharField()
    body = models.TextField()
    upload_date = models.DateTimeField()
```

### 1. ORM에서의 GROUP BY

시작에 앞서 우리는 한 유저(User) peter가 올린 공지(Announce)에 적힌 금액들의 총합을 알고싶다고 해보자.
django에서 aggregate를 통해 아래와 같이 작성할 수 있다.
```python
Announce.objects.filter(writer__userid='peter').aggregate(total_pay = Sum('pay'))
```

##### 1.1. 전체 유저들의 각각의 총 합 구하기
한 유저가 아닌 모든유저들의 각각의 pay의 총합을 알고 싶다고 해보자.

SQL상에서의 슈도코드는 아래와 같이 될것이다.
```sql
SELECT user.userid, SUM(announce.pay) FROM announce LEFT JOIN user on announce.writer_id = user.id 
GROUP BY user.userid
```

이 쿼리를 ORM에서 이용해보자.
Django내에서 values뒤에 annotate가 오면 values가 GROUP BY의 역할을 한다.
즉 아래와 같은 코드가 나오게 된다.
```python
q = Announce.objects.values('writer__userid').annotate(total_pay=Sum('pay'))
```

##### 1.2. 날짜별 전체 유저들의 총 합 구하기
날짜별로 각 유저들 본인이 올린 공고들의 총합을 구하려면 Group By로 upload_date와 userid 두가지 그룹묶음(?)을 동시에 적용하면된다.

```python
q1 = Announce.objects.extra(select={
        'date' : 'DATE(upload_date)'
    }).values('writer__userid', 'date').annotate(total_pay=Sum('pay'))
```

##### 1.2.1. 그룹화되지않은 값 SELECT했을시(?)
위의 예제를 보고 단순히 values는 GROUP BY annotate를 SELECT에 들어갈 그룹화 된 컬럼이라 생각했었다.
MySQL에서 그룹화 한 후, SELECT에 그룹화 할 수 없는 값들이 들어가면 오류가 난다.
######하지만 Django annotate에 그룹화 할 수 없는 값들이 들어가도 오류가 나지 않았다.
그냥 그 값도 GROUP BY에 추가한 후, 예상과 다른 값을 뱉는 것을 확인 할 수 있었다.

아래코드는 userid, date, body에대해 GROUP BY를 실행한다.
```python
q2 = Announce.objects.extra(select={
        'date' : 'DATE(upload_date)'
    }).values('writer__userid', 'date').annotate(total_pay=Sum('pay'), body=F('body'))
```


### 2. ORM에서의 HAVING
GROUP BY를 거쳐 나온 결과값에 필터를 적용하는 HAVING절을 장고에서는 어떻게 사용하는지 보자.

##### 2.1. filter을 이용한 HAVING
1.2.의 q1에 대해 total_pay가 80000원이 넘는 것만 결과로 뽑아내고 싶다고 해보자.
WHERE절에 했던것처럼 filter을 적용하면 쉽게 적용된다.
```python
q1 = Announce.objects.extra(select={
        'date' : 'DATE(upload_date)'
    }).values('writer__userid', 'date').annotate(total_pay=Sum('pay'))
q1.filter(total_pay__gt = 80000)
```

### 3. ORM에서의 CASE ~ WHEN문
SQL을 사용하다보면 프로그래밍의 IF ELIF같은 조건이 필요할때 CASE ~WHEN문을 사용한다.
만약 어드민에 대해서 할인율 100%적용하고 싶어서 할인율 필드를 만들고 싶다면 SQL은 아래와 같이 구성될 것이다.

```sql
SELECT announce.pay,
CASE
    WHEN user.is_admin = TRUE THEN "100%"
    ELSE "0%"
END AS "discount_rate"
FROM announce LEFT JOIN user
on announce.writer_id = user.id
```

파이썬에서는 ```from django.db.models import Case, When```를 이용해 아래와 같이 구성할 수 있다.
```python
Announce.objects.annotate(
    discount_rate = Case(
        When(
            writer__is_admin = True,
            then=Value('100%')
        ),
        default=Value('0%'),
        output_field=CharField()
    )
).values('pay','discount_rate')
```