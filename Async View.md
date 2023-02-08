# Async View

Django의 Async view를 썻을때의 장점을 테스트해보자.

## 1. Sync View

5초간 sleep하는 Blocking IO를 가진 SyncSleep클래스와 바로 결과를 반환해주는 SyncNoSleep클래스 두개를 만들어주자.

```python
class SyncSleep(View):
    def get(self, request, *args, **kwargs):
        print("START PID", os.getpid(), "TID", threading.get_native_id())
        time.sleep(5)
        return HttpResponse('sleep')
        

class SyncNoSleep(View):
    def get(self, request, *args, **kwargs):
        print("START PID", os.getpid(), "TID", threading.get_native_id())
        return HttpResponse('no sleep')
```

### 테스트서버 실행 (Multithreading 끄기)

Django는 자동으로 요청에 대해 multithreading을 지원한다. 

테스트를 위해 아래의 명령어로 runserver시 쓰레딩을 사용하지 않도록 해주자.

```sh
./manage.py runserver --nothreading
```

지금 킨 웹 어플리케이션 서버는 단일프로세스 단일 스레드로 동작하도록 한다.

### Test

SyncSleep 클래스로 request 한개 SyncNoSleep클래스로 request를 한개 보내보자.

![스크린샷 2023-02-08 오후 9 29 52](https://user-images.githubusercontent.com/30296115/217530086-df465125-1746-4892-84fb-f9fd49cff125.png)

첫번째 sleep request는 5초가 걸린다.

![스크린샷 2023-02-08 오후 9 30 26](https://user-images.githubusercontent.com/30296115/217530193-59464f69-318f-4aa6-80f5-e4529543dbba.png)

두번째 request도 sleep을 보내고 2초 후에 보내니 아직 첫번재 request가 처리되지않아 3초간 response하지 못하는걸 확인할 수 있었다.



## 2. Async View

Async view를 사용하면 위의처럼 긴 I/O작업시간이 발생했을때 다른 작업을 효율적으로 처리할 수 있다.

Async view를 작성해보자.

```python
class AsyncSleep(View):
    async def get(self, request, *args, **kwargs):
        print("START PID", os.getpid(), "TID", threading.get_native_id())
        # await asyncio.create_task(custom_coro())
        await asyncio.sleep(5)
        return HttpResponse('sleep')
class AsyncNoSleep(View):
    async def get(self, request, *args, **kwargs):
        print("START PID", os.getpid(), "TID", threading.get_native_id())
        return HttpResponse('no sleep')

```



### 서버 실행

위에서 사용한 Django의 manage.py runserver를 이용한 개발서버는 async view를 지원하지 않는다.

따라서 uvicorn을 이용해서 asgi 서버를 켜줘야한다.

```sh
  uvicorn *.asgi:application --workers 1 --reload
```

위의 테스트와 같은 환경을 구성하기 위해 프로세스를 하나만 사용하기 위해 workers를 1로 지정해주자.



### Test

위와 같은 순서대로 request를 만들어서 던져보자.

- 첫번째 요청(sleep)

![스크린샷 2023-02-08 오후 9 39 36](https://user-images.githubusercontent.com/30296115/217532124-dd4dcecc-ba80-4a9d-8646-3547616e04a9.png)

- 두번째 요청(no sleep)

![스크린샷 2023-02-08 오후 9 40 06](https://user-images.githubusercontent.com/30296115/217532225-d1078e7f-e5fc-47d8-b039-091f052cd15b.png)

sync view일때와 다르게 첫번째 요청의 I/O작업이 두번째 요청을 막고있지 않기때문에 두번째요청이 매우빠르게 실행된 것을 볼수있었다.

uvicorn에서는 --nothreading옵션이 없어서 혹시 다른 쓰레드에서 실행된게 아닌가 해서 print로 pid와 tid를 출력해보니 같은쓰레드에서 실행된것도 확인 할 수 있었다.

![스크린샷 2023-02-08 오후 9 42 07](https://user-images.githubusercontent.com/30296115/217532650-2643c8c2-937a-4d97-b08e-8e82577e4105.png)

## 3. 그 외..

위에서는 Sync view/Async view를 사용했을때 긴 IO의 영향을 극단적으로 확인해보고 싶어서 1프로세스 1스레드에서 실행되는 상황으로 테스트를했다. 사실 Sync view를 사용해도 멀티스레딩을 사용하기 때문에 위와같이 극단적인 상황이 나오지는 않는다.

