# Reference class

There are four distinct forms of references in the JVM, and indeed many of these apply to other garbage collected languages.

- Strong references
- Soft references
- Weak references
- Phantom references

It's important to know the differences, what affect they have on the collector and when you should be using them.

참고:
[Java Reference와 GC](https://d2.naver.com/helloworld/329631)

## Strong reference

Strong references never get collected

Strong References는 절대 GC의 대상이 되지 않습니다.

`new` 키워드로 생성된 instance가 reference class(SoftReference/WeakReference/PhantomReference)에 의한 캡슐화 없이 변수에 참조되는 경우 이 참조를 Strong reference 라고 합니다.

```java
public class ClassStrong {

    public static class Referred {
        // WARNING: Object.finalize() is deprecated since java 9.
        //          Consider AutoCloseable.close()
        protected void finalize() {
            System.out.println("Good bye cruel world");
        }
    }

    public static void collect() throws InterruptedException {
        System.out.println("Suggesting collection");
        System.gc();
        System.out.println("Sleeping");
        Thread.sleep(5000);
    }

    public static void main(String args[]) throws InterruptedException {
        System.out.println("Creating strong references");

        // This is now a strong reference.
        // The object will only be collected if all references to it disappear.
        Referred strong = new Referred();

        // Attempt to claim a suggested reference.
        ClassStrong.collect();

        System.out.println("Removing reference");
        // The object may now be collected.
        strong = null;
        ClassStrong.collect();

        System.out.println("Done");
    }
}
```

## Soft reference

[Soft references](http://download.oracle.com/javase/1.4.2/docs/api/java/lang/ref/SoftReference.html) only get collected if the JVM absolutely needs the memory.
This makes them excellent for implementing object cache's.

- GC 기준
  - GC timing + Unused Memory Size
  - JVM에 할당된 힙 메모리가 거의 남지 않았을 경우 GC가 됩니다.
  - Unused Memory의 크기에 따라 GC의 대상이 될지 결정 됩니다.

`new` 키워드로 생성된 instance가 SoftReference class에 의해 캡슐화 된 후 변수에 참조되는 경우 이 참조를 Soft reference 라고 합니다.

따라서 원문에서는 캐쉬(cache)로 사용하면 좋다고 언급하지만, 사실 soft reference는 캐쉬로 사용하면 안됩니다. 그 상세한 이유는 나중에 JVM 메모리에 대한 주제로 다룰때 언급하겠습니다. 다만 간략하게 설명하자면 soft reference로 인해 메모리에 GC threshold 까지 계속 차있는 상태가 유지되면서 훨씬 더 잦은 GC를 발생시킵니다. 그리고 안드로이드 같이 강력하게 GC에 대응하도록 구성된 경우에는 weak reference와 동작하는게 크게 다르지 않습니다. 메모리가 적은 모바일 기기 특성상 그냥 다 GC 처리됩니다.

```java
import java.lang.ref.SoftReference;
import java.util.ArrayList;
import java.util.List;

public class ClassSoft {

    public static class Referred {
        // WARNING: Object.finalize() is deprecated since java 9.
        //          Consider AutoCloseable.close()
        protected void finalize() {
            System.out.println("Good bye cruel world");
        }
    }

    public static void collect() throws InterruptedException {
        System.out.println("Suggesting collection");
        System.gc();
        System.out.println("Sleeping");
        Thread.sleep(5000);
    }

    public static void main(String args[]) throws InterruptedException {
        System.out.println("Creating soft references");

        // This is now a soft reference.
        // The object will be collected only if no strong references exist and the JVM really needs the memory.
        Referred strong = new Referred();
        SoftReference<Referred> soft = new SoftReference<Referred>(strong);

        // Attempt to claim a suggested reference.
        ClassSoft.collect();

        System.out.println("Removing reference");
        // The object may but highly likely wont be collected.
        strong = null;
        ClassSoft.collect();

        System.out.println("Consuming heap");
        try
        {
            // Create lots of objects on the heap
            List<ClassSoft> heap = new ArrayList<ClassSoft>(100000);
            while(true) {
                heap.add(new ClassSoft());
            }
        }
        catch (OutOfMemoryError e) {
            // The soft object should have been collected before this
            System.out.println("Out of memory error raised");
        }

        System.out.println("Done");
    }
}
```

## Weak reference

[Weak references](http://download.oracle.com/javase/1.4.2/docs/api/java/lang/ref/WeakReference.html) only get collected if no other object references it except the weak references.
This makes them perfect for keeping meta data about a particular object for the life time of the object.

- GC 기준
  - GC timing
  - GC 수행될 때 항상 GC의 대상이 됩니다.

new 키워드로 생성된 instance가 WeakReference class에 의해 캡슐화 된 후 변수에 참조되는 경우 이 참조를 Weak reference 라고 합니다.

명시적으로 weak reference를 사용함으로써 해당객체가 GC 되도록 유도할 수 있습니다.
가장 흔하게 쓰는 reference class 입니다.

```java
import java.lang.ref.WeakReference;
import java.util.ArrayList;
import java.util.List;

public class ClassWeak {

    public static class Referred {
        // WARNING: Object.finalize() is deprecated since java 9.
        //          Consider AutoCloseable.close()
        protected void finalize() {
            System.out.println("Good bye cruel world");
        }
    }

    public static void collect() throws InterruptedException {
        System.out.println("Suggesting collection");
        System.gc();
        System.out.println("Sleeping");
        Thread.sleep(5000);
    }

    public static void main(String args[]) throws InterruptedException {
        System.out.println("Creating weak references");

        // This is now a weak reference.
        // The object will be collected only if no strong references.
        Referred strong = new Referred();
        WeakReference<Referred> weak = new WeakReference<Referred>(strong);

        // Attempt to claim a suggested reference.
        ClassWeak.collect();

        System.out.println("Removing reference");
        // The object may be collected.
        strong = null;
        ClassWeak.collect();

        System.out.println("Done");
    }
}
```

## Phantom reference

[PhantomReference](https://docs.oracle.com/javase/8/docs/api/java/lang/ref/PhantomReference.html) are objects that can be collected whenever the collector likes. The object reference is appended to a ReferenceQueue and you can use this to clean up after a garbage collection. This is an alternative to the finalize() method and is slightly safer because the finalize() method may ressurect the object by creating new strong references. The PhantomReference however cleans up the object and enqueues the reference object to a ReferenceQueue that a class can use for clean up.

Phantom reference는 strong-reachable이 제거되어 gc의 대상이 된 referent가 gc에 의해 메모리 회수를 당하지 않도록 막는다.
Phantom reference의 get() 메소드가 항상 null을 반환 함으로써 gc가 referent를 획득하지 못하도록 하여 referent가 gc의 입장에서 `죽었지만 메모리를 회수할 수 없는 유령 instance`가 되도록 한다.

PhantomReference의 reference는 자신의 선언에 Object.finalize() method를 override 하지 말아야 한다. 만약 override 한다면 garbage collector는 ReferenceQueue에 queueing 하는대신 finalization을 수행하고 메모리를 회수해 버린다. ***`Object.finalize()는 java9 부터 deprecated 됩니다. java9부터는 finalize() override 대신 AutoCloseble.close()를 구현하는 것을 권장합니다.`***
Phantom reference가 ReferenceQueue에 담겼다는 의미는 그 referent가 모든 strong-reachable이 제거된 gc의 대상이 되었다는 것을 의미한다. ReferenceQueue에 담긴 Phnantom Reference instance는 추후에 반드시 Queue에 꺼낸 후 파이널라이즈를 대신할 리소스 정리 작업을 진행하고 reference.clear() 호출을 통해 referent가 메모리 회수의 대상이 되도록 해야한다.

garbage collector가 PhantomReference의 referent를 finalization하지 못하는 이유는 referent가 finalize()를 override하지 않았기 때문이고 메모리를 회수하지 못하는 것은 PhantomReference의 get() 메소드가 항상 null을 반환함으로써 gc가 PhantomReference의 referent를 획득 할 수 없기 때문이다.

GC가 객체를 처리하는 순서는 항상 다음과 같다.

  1. soft references
  2. weak references
  3. 파이널라이즈 //phantom reference의 referent는 finalize()를 override하지 않으므로 이 과정은 건너뛰게 된다.
  4. phantom references // garbage collector는 phantom reference들을 ReferenceQueue에 담게 된다. Phantom reference가 ReferenceQueue에 담겼다는 의미는 그 referent가 모든 strong-reachable이 제거된 gc의 대상이 되었다는 것을 의미한다. ReferenceQueue에 담긴 Phnantom Reference instance는 추후에 반드시 Queue에 꺼낸 후 파이널라이즈를 대신할 리소스 정리 작업을 진행하고reference.clear() 호출을 통해 referent가 메모리 회수의 대상이 되도록 해야한다.
  5. 메모리 회수 // Phantom reference에 참조된 referent를 제외한 파이널라이즈가 완료된 모든 referent의 메모리는 회수된다.

```java
import java.lang.ref.PhantomReference;
import java.lang.ref.Reference;
import java.lang.ref.ReferenceQueue;
import java.util.HashMap;
import java.util.Map;

public class ClassPhantom {
    public static class Referred {
        // Note that if there is a finalize() method
        // PhantomReferences don't get appended to a ReferenceQueue
        // protected void finalize() {
        //   System.out.println("Good bye cruel world");
        // }
    }

    public static void collect() throws InterruptedException {
        System.out.println("Suggesting collection");
        System.gc();
        System.out.println("Sleeping");
        Thread.sleep(5000);
    }

    public static void main(String args[]) throws InterruptedException {
        System.out.println("Creating phantom references");

        // The reference itself will be appended to the dead queue for clean up.
        ReferenceQueue<Referred> dead = new ReferenceQueue<>();

        // This map is just a sample we might use to locate resources we need to clean up.
        Map<Reference<Referred>,String> cleanUpMap = new HashMap<>();

        // This is now a phantom reference.
        // The object will be collected only if no strong references.
        Referred strong = new Referred();
        PhantomReference<Referred> phantom = new PhantomReference<>(strong, dead);
        cleanUpMap.put(phantom, "You need to clean up some resources, such as me!");

        // remove strong reference so that referent to be target of garbage collection
        strong = null;

        // The object may now be collected,
        // so that reference have got appended into a dead(ReferenceQueue) by garbage collector
        ClassPhantom.collect();

        // Check for queued PhantomReference
        Reference<? extends Referred> reference = dead.poll();
        if (reference != null) {
            // remove references against queued PhantomReference
            System.out.println(cleanUpMap.remove(reference));
            
            //remove phantom reachable so that referent's memory can be retrieved by gc
            reference.clear();
        }

        System.out.println("Done");
    }
}
```

### ReferenceQueue

You saw me use the reference queue class in the previous example. A ReferenceQueue instance can be supplied as an argument to SoftReference, WeakReference or PhantomReference. When an object is collected the reference instance itself will be enqueued to the supplied [ReferenceQueue](http://download.oracle.com/javase/1.5.0/docs/api/java/lang/ref/ReferenceQueue.html). This allows you to perform clean up operations on the object. This is useful if you are implementing any container classes that you want to contain a Soft, Weak or Phantom reference and some associated data because you can get notified via the ReferenceQueue which Reference was just collected.

referent가 gc 시점에 garbage collector에 의해 gc 대상이 되면 finalizaion 되는 대신에 그 Phantom reference를 따로 담아놓는 Queue.

reference?
    PhantomReference instance

referenct?
    참조 대상, PhantomReference에 의해 참조되는 원본 instance

API 설명) garbage collector가 그들의 referent(참조대상)들이 회수될 수 있다고(reclaimed) 결정한 이 후에
`ReferenceQueue`에 enqueue되는 Phantom reference object들 입니다.

Phantom reference들은 Java finalization mechanism이 가능한 것 보다 좀 더 유연한 방식으로 사전 정리 조치(pre-mortem cleanup actions)를 스케줄링하기 위해 가장 자주 사용된다.

만약 garbage collector가 어느 특정 시점에 phantom reference의 referent(참조대상)이 phantom reachable이라고 결정한다면,
해당 시점 또는 조금 더 후에 garbage collector는 referece를 <ReferenceQueue>에 enqueue할 것이다.

회수 가능한 object가 그대로 남아있도록 보장하기 위해서,
phantom reference의 referent(참조대상)은 garbage collector에 의해 획득 되지 않을 수 도 있다.
: phantom reference의 get() method는 항상 null을 반환함으로써

soft와 weak reference와는 달리, phantom reference는 enqueue 됨으로써 garbage collector에 의해 자동으로 clear되지 않게 된다.
phantom reference에 의해 reachable인 object는 모든 reference들이 clear되거나 그들 스스로 unreachable이 될 때 까지 clear 되지 않고 남아있을 것이다.

## WeakHashMap class

There is also a convience [WeakHashMap](http://download.oracle.com/javase/1.4.2/docs/api/java/util/WeakHashMap.html) that wraps all keys by a weak reference. Allowing you to easily store meta data against an object and have the map entry including the meta data removed and collected when the original object itself is unreachable.

weak reference를 써서 메모리에 좀 신경썼다 하더라도 hashmap으로 잡혀있는 경우가 꽤 있습니다.
따라서 이런 경우를 위해 WeakHashMap이 따로 구현되어 있습니다.
직접 weak reference를 갖는 hashMap을 만들어 써도 같은 효과 가능.

```java
import java.util.Map;
import java.util.WeakHashMap;

public class ClassWeakHashMap {

    public static class Referred {
        // WARNING: Object.finalize() is deprecated since java 9.
        //          Consider AutoCloseable.close()
        protected void finalize() {
            System.out.println("Good bye cruel world");
        }
    }

    public static void collect() throws InterruptedException {
        System.out.println("Suggesting collection");
        System.gc();
        System.out.println("Sleeping");
        Thread.sleep(5000);
    }

    public static void main(String args[]) throws InterruptedException {
        System.out.println("Creating weak references");

        // This is now a weak reference.
        // The object will be collected only if no strong references.
        Referred strong = new Referred();
        Map<Referred, String> metadata = new WeakHashMap<Referred, String>();
        metadata.put(strong, "WeakHashMap's make my world go around");

        // Attempt to claim a suggested reference.
        ClassWeakHashMap.collect();
        System.out.println("Still has metadata entry? " + (metadata.size() == 1));
        System.out.println("Removing reference");
        // The object may be collected.
        strong = null;
        ClassWeakHashMap.collect();

        System.out.println("Still has metadata entry? " + (metadata.size() == 1));

        System.out.println("Done");
    }
}
```
