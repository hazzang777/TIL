# CompletableFuture

## 자바 Concurrent 프로그래밍 소개

- Concurrent 소프트웨어
    - 동시에 여러 작업을 할 수 있는 소프트웨어
- 자바에서 지원하는 Concurrent 프로그래밍
    - 멀티프로세싱 (ProcessBuilder)
    - 멀티쓰레드
- 자바 멀티쓰레드 프로그래밍
    - Thread/Runnable
    - 쓰레드의 순서는 보장 못한다.

```java
public class App {
    public static void main(String[] args) throws InterruptedException {
        MyThread myThread = new MyThread();
        myThread.start();

        System.out.println("Hello");
    }

    static class MyThread extends Thread {
        @Override
        public void run() {
            System.out.println("Hello: " + Thread.currentThread().getName());
        }
    }
}
```

Runnable 구현 또는 람다

```java
public class App {
    public static void main(String[] args) throws InterruptedException {
        Thread thread = new Thread(() -> System.out.println("Thread: " + Thread.currentThread().getName()));
        thread.start();

        System.out.println("Hello: " + Thread.currentThread().getName());
    }
}
```

- 쓰레드 주요 기능

→ 현재 쓰레드 멈춰두기 (sleep) : 다른 쓰레드가 처리할 수 있도록 기회를 주지만 그렇다고 락을 놔주진 않는다. (잘못하면 데드락이 걸린다.)

```java
public class App {
    public static void main(String[] args) throws InterruptedException {
        Thread thread = new Thread(() -> {
            try {
                Thread.sleep(1000L);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println("Thread: " + Thread.currentThread().getName());
        });
        thread.start();
        System.out.println("Hello: " + Thread.currentThread().getName());
    }
}
```

→ 다른 쓰레드 깨우기 (interrupt) : 다른 쓰레드를 깨워서 interruptedException을 발생시킨다. 그 에러가 발생했을 때 할 일은 코딩하기 나름. 종료 시킬 수도 있고 계속 하던 일 지속할 수 있다.

```java
public class App {
    public static void main(String[] args) throws InterruptedException {
        Thread thread = new Thread(() -> {
            while(true) {
                System.out.println("Thread: " + Thread.currentThread().getName());
                try {
                    Thread.sleep(1000L);
                } catch (InterruptedException e) {
                    System.out.println("exit!");
                    return;
                }
            }
        });
        thread.start();
        
        System.out.println("Hello: " + Thread.currentThread().getName());
        Thread.sleep(3000L);
        thread.interrupt();
    }
}
```

→ 다른 쓰레드 기다리기 (join) : 다른 쓰레드가 끝날 때까지 기다린다.

```java
public class App {
    public static void main(String[] args) throws InterruptedException {
        Thread thread = new Thread(() -> {
                System.out.println("Thread: " + Thread.currentThread().getName());
                try {
                    Thread.sleep(3000L);
                } catch (InterruptedException e) {
                    throw new IllegalStateException(e);
                }
        });
        thread.start();

        System.out.println("Hello: " + Thread.currentThread().getName());
        thread.join();
        System.out.println(thread + " is finished");
    }
}
```

<aside>
💡 쓰레드가 두개만 되어도 많이 복잡해진다.

</aside>

→ 그래서 Executors가 나왔다.

## Executors

- 고수준 Concurrency 프로그래밍
    - 쓰레드를 만들고 관리하는 작업을 애플리케이션에서 분리
    - 그런 기능을 Executors에게 위임

- Executors가 하는 일
    - 쓰레드 만들기: 애플리케이션이 사용할 쓰레드 풀을 만들어 관리
    - 쓰레드 관리 : 쓰레드 생명 주기를 관리
    - 작업 처리 및 실행: 쓰레드로 실행할 작업을 제공할 수 있는 API 제공

- 주요 인터페이스
    - Executor: execute(Runnable)
    - ExecutorService: Executor 상속 받은 인터페이스, Callable 실행 가능, Executor를 종료 시키거나, 여러 Callable을 동시에 실행하는 등의 기능을 제공
        - 다음 작업이 들어올때 까지 계속 대기하기 때문에 프로세스가 죽지 않는다. → 명시적으로 shutdown 해야 한다.
    - ScheduledExecutorService: ExecutorService를 상속 받은 인터페이스로 특정 시간 이후에 또는 주기적으로 작업을 실행할 수 있음
    
    ```java
    public class App {
        public static void main(String[] args) throws InterruptedException {
            ExecutorService executorService = Executors.newSingleThreadExecutor();
            executorService.submit(() -> {
                System.out.println("Thread " + Thread.currentThread().getName());
            });
            executorService.shutdown(); // 처리중인 작업 기다렸다가 종료
            // executorService.shutdownNow();  당장 종료
        }
    }
    ```
    
    예)  2개의 쓰레드로 4개의 작업을 처리한다.
    
    쓰레드풀을 사용하는 이유 → 쓰레드 생성 비용을 줄여준다.
    
    ```java
    public class App {
        public static void main(String[] args) throws InterruptedException {
            ExecutorService executorService = Executors.newFixedThreadPool(2); // 쓰레드풀에 2개의 쓰레드
            executorService.submit(getRunnable("hardy"));
            executorService.submit(getRunnable("dev"));
            executorService.submit(getRunnable("java"));
            executorService.submit(getRunnable("spring"));
            executorService.shutdown(); // 처리중인 작업 기다렸다가 종료
        }
    
        private static Runnable getRunnable(String message) {
            return () -> System.out.println(message + Thread.currentThread().getName());
        }
    }
    ```
    
- Fork / Join 프레임워크
    - ExecutorService의 구현체로 손쉽게 멀티 프로세서를 활용할 수 있게끔 도와줌.

## Callable과 Future

- Callable
    - Runnable과 유사하지만 작업의 결과를 받을 수 있다.
- Future
    - 비동기적인 작업의 현재 상태를 조회하거나 결과를 가져올 수 있다.

1. 결과 가져오기 get()

```java
public class App {
    public static void main(String[] args) throws ExecutionException, InterruptedException {
        ExecutorService executorService = Executors.newSingleThreadExecutor();

        Callable<String> hello = () -> {
            Thread.sleep(2000L);
            return "Hello";
        };

        Future<String> submit = executorService.submit(hello);
        System.out.println("Started!");

        submit.get(); // 결과값을 가져올때 까지 기다린다. -> 블록킹 콜

        System.out.println("End!!");
        executorService.shutdown(); // 처리중인 작업 기다렸다가 종료
    }
}
```

- 블록킹 콜이다.
- 타임아웃(최대한으로 기다릴 시간)을 설정 가능

```java
submit.get(10, TimeUnit.SECONDS);
```

1. 작업 상태 확인하기 isDone()
    1. 완료 했으면 true 아니면 false 리턴

```java
public class App {
    public static void main(String[] args) throws ExecutionException, InterruptedException, TimeoutException {
        ExecutorService executorService = Executors.newSingleThreadExecutor();

        Callable<String> hello = () -> {
            Thread.sleep(3000L);
            return "Hello";
        };

        Future<String> helloFuture = executorService.submit(hello);
        System.out.println(helloFuture.isDone());
        System.out.println("Started!");

        helloFuture.get();

        System.out.println(helloFuture.isDone());
        System.out.println("End!!");
        executorService.shutdown(); // 처리중인 작업 기다렸다가 종료
    }
}
```

1. 작업 취소하기 cancel()
    1. 취소 했으면 true 못했으면 false를 리턴함
    2. parameter로 true를 전달하면 현재 진행중인 쓰레드를 interrupt하고 그러지 않으면 현재 진행중인 작업이 끝날때까지 기다림
    3. cancel()을 하면 get()으로 가져올 수 없다. isDone()은 무조건 true

```java
helloFuture.cancel(true);
```

1. 여러 작업 동시에 실행하기 invokeAll()
    1. 동시에 실행한 작업 중에 제일 오래 걸리는 작업 만큼 시간이 걸린다.
        1. 작업들 중에 먼저 끝난 것이 있다하더라도 invokeAll은 모든 작업이 끝날때까지 기다리기 때문에 작업 중에서 가장 오래 걸리는 작업 만큼 시간이 걸림

```java
public class App {
    public static void main(String[] args) throws ExecutionException, InterruptedException, TimeoutException {
        ExecutorService executorService = Executors.newSingleThreadExecutor();

        Callable<String> hello = () -> {
            Thread.sleep(2000L);
            return "Hello";
        };

        Callable<String> java = () -> {
            Thread.sleep(3000L);
            return "Java";
        };

        Callable<String> hardy = () -> {
            Thread.sleep(1000L);
            return "hardy";
        };

        List<Future<String>> futures = executorService.invokeAll(Arrays.asList(hello, java, hardy));
        for (Future<String> f : futures) {
            System.out.println(f.get());
        }

        executorService.shutdown(); // 처리중인 작업 기다렸다가 종료
    }
}
```

1. 여러 작업 중에 하나라도 먼저 응답이 오면 끝내기 invokeAny()
    1. 동시에 실행한 작업 중에 제일 짧게 걸리는 작업 만큼 시간이 걸림
    2. 블록킹 콜

```java
public class App {
    public static void main(String[] args) throws ExecutionException, InterruptedException, TimeoutException {
        ExecutorService executorService = Executors.newFixedThreadPool(3);

        Callable<String> hello = () -> {
            Thread.sleep(2000L);
            return "Hello";
        };

        Callable<String> java = () -> {
            Thread.sleep(3000L);
            return "Java";
        };

        Callable<String> hardy = () -> {
            Thread.sleep(1000L);
            return "hardy";
        };

        String s = executorService.invokeAny(Arrays.asList(hello, java, hardy));
        System.out.println(s); // hardy
        executorService.shutdown(); // 처리중인 작업 기다렸다가 종료
    }
}
```

## CompletableFuture 1

자바에서 비동기 프로그래밍을 가능하게하는 인터페이스

→ Future를 사용해서도 어느정도 가능했지만 하기 힘든게 많음

→ Future의 단점

- Future를 외부에서 완료 시킬 수 없다. 취소하거나, get() 타임 아웃 설정할 수 있다.
- 블로킹 코드(get())을 사용하지 않고서는 작업이 끝났을 때 콜백을 실행할 수 없다.
- 여러 Future를 조합할 수 없다.
- 예외 처리용 API를 제공하지 않는다.

CompletableFuture

- Future를 구현
- CompletionStage를 구현

```java
public class App {
    public static void main(String[] args) throws ExecutionException, InterruptedException, TimeoutException {
        CompletableFuture<String> future = CompletableFuture.completedFuture("hardy");
        System.out.println(future.get()); // hardy
    }
}
```

비동기로 작업 실행

1. 리턴값이 없는 경우 : runAsync()
2. 리턴값이 있는 경우 : supplyAsync()
3. 원하는 Executor(쓰레드풀)을 사용해서 실행 가능
    1. 기본은 ForkJoinPool.commonPool()

(1) runAsync()

```java
public class App {
    public static void main(String[] args) throws ExecutionException, InterruptedException, TimeoutException {
        CompletableFuture<Void> future = CompletableFuture.runAsync(() -> {
            System.out.println("Hardy " + Thread.currentThread().getName());
        });
        future.get(); // Hardy ForkJoinPool.commonPool-worker-3
    }
}
```

(2) supplyAsync()

```java
public class App {
    public static void main(String[] args) throws ExecutionException, InterruptedException, TimeoutException {
        CompletableFuture<String> future = CompletableFuture.supplyAsync(() -> {
            System.out.println("Hello " + Thread.currentThread().getName());
            return "hardy";
        });
        System.out.println(future.get());
    }
}
```

콜백 제공하기 : `Future는 get() 호출하기전에 콜백 지정 불가, 하지만 CompletableFuture는 가능`

1. thenApply(Function) : 리턴 값을 받아서 다른 값으로 바꾸는 콜백
2. thenAccept(Consumer) : 리턴값을 또 다른 작업을 처리하는 콜백 (리턴 X)
3. thenRun(Runnable) : 리턴값 받지 않고 다른 작업을 처리하는 콜백
4. 콜백 자체를 또 다른 쓰레드에서 실행 가능

예) 콜백 예시

```java
public class App {
    public static void main(String[] args) throws ExecutionException, InterruptedException, TimeoutException {
        CompletableFuture<String> future = CompletableFuture.supplyAsync(() -> {
            System.out.println("Hello " + Thread.currentThread().getName());
            return "hardy";
        }).thenApply((s) -> {
            System.out.println(Thread.currentThread().getName());
            return s.toUpperCase();
        });

        System.out.println(future.get());
    }
}
```

예) 쓰레드 지정

```java
public class App {
    public static void main(String[] args) throws ExecutionException, InterruptedException, TimeoutException {
        ExecutorService executorService = Executors.newFixedThreadPool(4);
        CompletableFuture<String> future = CompletableFuture.supplyAsync(() -> {
            System.out.println("Hello " + Thread.currentThread().getName());
            return "hardy";
        }, executorService).thenApplyAsync((s) -> { // 쓰레드 지정
            System.out.println(Thread.currentThread().getName());
            return s.toUpperCase();
        },executorService); // 콜백에도 쓰레드 지정 가능 xxxAsync()

        future.get();
        executorService.shutdown();

    }
}
```

- 기본은 Fork / Join Thread Pool → 원한다면 Executor에서 쓰레드풀 사용 가능

## CompletableFuture 2

조합하기

1. thenCompose(): 두 작업이 서로 이어서 실행하도록 조합
2. thenCombine(): 두 작업을 독립적으로 실행하고 둘 다 종료 했을 때 콜백 실행
3. allOf(): 여러 작업을 모두 실행하고 모든 작업 결과에 콜백 실행
4. anyOf(): 여러 작업 중에 가장 빨리 끝난 하나의 결과에 콜백 실행

(1) thenCompose

```java
public class App {
    public static void main(String[] args) throws ExecutionException, InterruptedException, TimeoutException {
        CompletableFuture<String> hello = CompletableFuture.supplyAsync(() -> {
            System.out.println("Hello " + Thread.currentThread().getName());
            return "Hello";
        });

        CompletableFuture<String> future = hello.thenCompose(App::getWorld);
        System.out.println(future.get());
    }

    private static CompletableFuture<String> getWorld(String messege) {
        return CompletableFuture.supplyAsync(() -> {
            System.out.println("World " + Thread.currentThread().getName());
            return messege + "World";
        });
    }
}
```

(2) thenCombine()

```java
public class App {
    public static void main(String[] args) throws ExecutionException, InterruptedException, TimeoutException {
        CompletableFuture<String> hello = CompletableFuture.supplyAsync(() -> {
            System.out.println("Apple " + Thread.currentThread().getName());
            return "Apple";
        });

        CompletableFuture<String> world = CompletableFuture.supplyAsync(() -> {
            System.out.println("MS " + Thread.currentThread().getName());
            return "MS";
        });
				// h는 future의 결과값 , w는 world의 결과값 / BiFunction 사용
        CompletableFuture<String> future = hello.thenCombine(world, (h, w) -> h + " " + w);
        System.out.println(future.get());
    }
}
```

(3) CompletableFuture.allOf()

```java
public class App {
    public static void main(String[] args) throws ExecutionException, InterruptedException, TimeoutException {
        CompletableFuture<String> hello = CompletableFuture.supplyAsync(() -> {
            System.out.println("Apple " + Thread.currentThread().getName());
            return "Apple";
        });
        CompletableFuture<String> world = CompletableFuture.supplyAsync(() -> {
            System.out.println("MS " + Thread.currentThread().getName());
            return "MS";
        });
        CompletableFuture<Void> future = CompletableFuture.allOf(hello, world)
                .thenAccept(System.out::println);
        future.get();
    }
}
```

(4) anyOf() : 여러 작업 중에 가장 빨리 끝난 하나의 결과에 콜백 실행

```java
public class App {
    public static void main(String[] args) throws ExecutionException, InterruptedException, TimeoutException {
        CompletableFuture<String> hello = CompletableFuture.supplyAsync(() -> {
            System.out.println("Apple " + Thread.currentThread().getName());
            return "Apple";
        });
        CompletableFuture<String> world = CompletableFuture.supplyAsync(() -> {
            System.out.println("MS " + Thread.currentThread().getName());
            return "MS";
        });
        CompletableFuture<String> future = CompletableFuture.anyOf(hello, world)
                .thenApply((s) -> {
                    System.out.println(s);
                    return s + "Success";
                });
        System.out.println(future.get());
    }
}
```

예외 처리

1. exceptionally(Function)
2. handle(BiFunction) : 정상적인 경우와 예외 경우 둘다 가능

(1) exceptionally(Function)

```java
public class App {
    public static void main(String[] args) throws ExecutionException, InterruptedException, TimeoutException {
        boolean throwError = true;

        CompletableFuture<String> hello = CompletableFuture.supplyAsync(() -> {
            if (throwError) {
                throw new IllegalArgumentException();
            }
            System.out.println("Hello " + Thread.currentThread().getName());
            return "Hello";
        }).exceptionally(ex -> {
            System.out.println(ex);
            return "Error!";
        });
        System.out.println(hello.get());
    }
}
```

(2) handle(BiFunction)

```java
public class App {
    public static void main(String[] args) throws ExecutionException, InterruptedException, TimeoutException {
        boolean throwError = true;

        CompletableFuture<String> hello = CompletableFuture.supplyAsync(() -> {
            if (throwError) {
                throw new IllegalArgumentException();
            }
            System.out.println("Hello " + Thread.currentThread().getName());
            return "Hello";
        }).handle((result, ex) -> {
            if (ex != null) {
                System.out.println(ex);
                return "Error";
            }
            return result;
        });
        System.out.println(hello.get());
    }
}
```