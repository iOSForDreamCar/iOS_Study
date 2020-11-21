### 11월 17일 CoreData 학습

오늘의 주제: Core Data

학습한 내용을 정리하거나 참고자료를 올려주세요😀

- 코어 데이터 튜토리얼
    
    - [https://www.raywenderlich.com/7569-getting-started-with-core-data-tutoria](https://www.raywenderlich.com/7569-getting-started-with-core-data-tutorial)
    - [https://zeddios.tistory.com/987](https://zeddios.tistory.com/987)

- Codegen에서의 Menu는 3가지가 있다.
    - Manual/None은 Class, Extension
    - Class Definition은 Class, Extension 필요없음
    - Category/Extension은 Class만 필요

<img width="218" alt="1" src="https://user-images.githubusercontent.com/46857148/99373055-505fdd80-2904-11eb-97d2-d107d12bb83a.png">
<img width="861" alt="2" src="https://user-images.githubusercontent.com/46857148/99373065-52c23780-2904-11eb-818f-49a49b954ae0.png">

```swift
// Core Data save

// Setting
let appDelegate = UIApplication.shared.delegate as! AppDelegate
let context = appDelegate.persistentContainer.viewContext

// 저장1
let person = Person(context: context)

person.age = 1
person.name = "name"
person.person = "person"

try? context.save()

// 저장2
let entity = NSEntityDescription.entity(forEntityName: "Person", in: context)
if let entity = entity {
    let model = NSManagedObject(entity: entity, insertInto: context)
    model.setValue(123, forKey: "age")
    model.setValue("n", forKey: "name")
    model.setValue("p", forKey: "person")
}
try? context.save()
```

```swift
// 출력
func fetchContact() {
    let appDelegate = UIApplication.shared.delegate as! AppDelegate
    let context = appDelegate.persistentContainer.viewContext
    do {
        let contact = try context.fetch(Person.fetchRequest()) as! [Person]
        contact.forEach {
            print($0.name)
        }
    } catch {
        print(error.localizedDescription)
    }
}
```

```swift
// 삭제
let request2: NSFetchRequest<Person> = Person.fetchRequest()
deleteAll(request: request2)

func deleteAll<T: NSManagedObject>(request: NSFetchRequest<T>) {
    let request: NSFetchRequest<NSFetchRequestResult> = T.fetchRequest()
    let delete = NSBatchDeleteRequest(fetchRequest: request)
    try? self.context.execute(delete)
}
```

* [NSPredicate CheetSheet](https://academy.realm.io/posts/nspredicate-cheatsheet/)




## 11월 20일

## CoreData란? 단일장치에서 데이터를 유지 또는 캐시, 실행 취소를 지원해줍니다.

- Core Data는 런타임에 개체 인스턴스를 관리하여 다음 기능을 제공 할 수 있습니다.
- Core Data는 객체를 저장소에 매핑하는 **세부 정보를 추상화**하여 데이터베이스를 직접 관리하지 않고도 Swift의 데이터를 쉽게 저장할 수 있습니다.

- [NSManagedObjectModel](https://developer.apple.com/documentation/coredata/nsmanagedobjectmodel)
    - 앱의 유형, 속성 및 관계를 설명하는 앱의 모델 파일

- [NSManagedObjectContext](https://developer.apple.com/documentation/coredata/nsmanagedobjectcontext)
    - 앱 유형의 인스턴스에 대한 변경 사항을 추적

- [NSPersistentStoreCoordinator](https://developer.apple.com/documentation/coredata/nspersistentstorecoordinator)
    - 스토어에서 앱 유형 의 인스턴스를 저장하고 가져 오는 인스턴스

- [NSPersistentContainer](https://developer.apple.com/documentation/coredata/nspersistentcontainer)
    - Model, Context 및 Store Coordinator를 한 번에 모두 설정

### Persistent Container 초기화

```swift
// 프로젝트 생성할 때 UserData를 선택한다면 자동으로 생성해준다.

lazy var persistentContainer: NSPersistentContainer = {        
        let container = NSPersistentContainer(name: "DataModel")
        container.loadPersistentStores { description, error in
            if let error = error {
                fatalError("Unable to load persistent stores: \(error)")
            }
        }
        return container
    }()
```

- 이제 컨테이너에 대한 참조를 사용자 인터페이스에 전달

### Persistent Container Reference를 View Controller에 전달

```swift
// Core Data를 가져오고 Persistent Container에 대한 참조를 보유 할 변수를 만듭니다.

class ViewController: UIViewController {

    var container: NSPersistentContainer!
    
    override func viewDidLoad() {
        super.viewDidLoad()
        guard container != nil else {
            fatalError("This view needs a persistent container.")
        }
        // The persistent container is available.
    }
}
```

- rootViewController에서 Container를 Persistent Container로 설정합니다.

```swift
func application(_ application: UIApplication, didFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey: Any]?) -> Bool {        
      if let rootVC = window?.rootViewController as? ViewController {
          rootVC.container = persistentContainer
      }        
      return true
}
```

- Persistent Container를 Next ViewController로 전달하려면 Containger변수 생성을 반복하고 prepare로 통해 전달합니다.

```swift
class ViewController: UIViewController {

    ...
    
    override func prepare(for segue: UIStoryboardSegue, sender: Any?) {
        if let nextVC = segue.destination as? NextViewController {
            nextVC.container = container
        }
    }
}
```

### Subclass the Persistent Container

- 하위 클래스는 **데이터 하위 집합을 반환하는 함수**와 **데이터를 디스크에 유지하기위한 호출**

```swift
import CoreData

class PersistentContainer: NSPersistentContainer {    

    func saveContext(backgroundContext: NSManagedObjectContext? = nil) {
        let context = backgroundContext ?? viewContext
        guard context.hasChanges else { return }
        do {
            try context.save()
        } catch let error as NSError {
            print("Error: \(error), \(error.userInfo)")
        }
    }    
}
```

 

### What are Concurrency type in Core Data?

- `[NSMainQueueConcurrencyType](https://developer.apple.com/documentation/coredata/nsmanagedobjectcontextconcurrencytype/1506566-mainqueueconcurrencytype)` 특히 Application Interface와 함께 사용하기 위한 것이며 애플리케이션의 Main queue에서만 사용할 수 있습니다.
- 
- `[NSPrivateQueueConcurrencyType](https://developer.apple.com/documentation/coredata/nsmanagedobjectcontextconcurrencytype/nsprivatequeueconcurrencytype)`구성은 초기에 자신의 큐를 만들고 그 큐에서 만 사용할 수 있습니다. 큐는 개인용이며 `[NSManagedObjectContext](https://developer.apple.com/documentation/coredata/nsmanagedobjectcontext)`인스턴스 내부에 있으므로 `[performBlock:](https://developer.apple.com/documentation/coredata/nsmanagedobjectcontext/1506578-performblock)`및 `[performBlockAndWait:](https://developer.apple.com/documentation/coredata/nsmanagedobjectcontext/1506364-performandwait)` method 를 통해서만 액세스 할 수 있습니다 .

### How Core Data behave with multi threading?

- Core Data는 단일 스레드에서 실행될 것으로 예상된다.
- NSManagedObject는 한 스레드에서 다른 스레드로 전달되지 않아야 한다.
    - 한 스레드에서 다른 스레드로 관리되는 개체를 전달해야 하는 경우 관리되는 객체의 ObjectID속성을 사용
- never share managed object contexts between threads
- 

### batch faulting and prefetching

[Performance](https://developer.apple.com/library/archive/documentation/Cocoa/Conceptual/CoreData/Performance.html)

### Entity에 대한 몇가지 속성 가져오는 방법

- [propertiesToFetch](https://developer.apple.com/documentation/coredata/nsfetchrequest/1506851-propertiestofetch)

### 고유 한 값을 가져 오는 방법

- iOS에서는 NSExpressionDescription객체를 사용하여 가져 오기 요청에 대한 함수를 지정하고 [setReturnsDistinctResults](https://developer.apple.com/documentation/coredata/nsfetchrequest/1506344-returnsdistinctresults) 고유 한 값을 반환
- [https://developer.apple.com/documentation/coredata/nsfetchedresultscontroller](https://developer.apple.com/documentation/coredata/nsfetchedresultscontroller)

### fetchedResultController

- Core Data 가져 오기 요청의 결과를 관리하고 사용자에게 데이터를 표시하는 데 사용하는 컨트롤러입니다.
- [https://developer.apple.com/documentation/coredata/nsfetchedresultscontroller](https://developer.apple.com/documentation/coredata/nsfetchedresultscontroller)

### 컨텍스트 동기화

- [https://cocoacasts.com/how-to-observe-a-managed-object-context](https://cocoacasts.com/how-to-observe-a-managed-object-context)

### NSFetchRequest

- NSFetchRequest는 Core Data에서 가져 오기를 담당하는 클래스
- 가져 오기 요청을 사용하여 제공된 기준, 개별 값 등을 충족하는 개체 집합을 가져올 수 있음

### NSPersistentContainer

- 영구 컨테이너는 애플리케이션 저장소를로드 한 컨테이너를 만들고 반환합니다. 저장소 생성에 실패 할 수있는 합법적 인 오류 조건이 있으므로이 속성은 선택 사항입니다.

### NSFetchedResultsController

- NSFetchedResultsController는 컨트롤러이지만 뷰 컨트롤러는 아닙니다 .
- 테이블 뷰를 Core Data가 지원하는 데이터 소스와 동기화하는 데 필요한 많은 코드를 추상화

# **SQLite 제한이란 무엇입니까?**

- 테이블 간의 관계를 정의해야합니다. 모든 테이블의 스키마를 정의하십시오.
- 데이터를 가져 오기 위해 수동으로 쿼리를 작성해야합니다.
- 결과를 쿼리 한 다음이를 모델에 매핑해야합니다.
- 쿼리는 매우 빠릅니다

### Core 데이터의 삭제 규칙이란 무엇입니까?

- **Deny:** If there is at least one object at the relationship destination (employees), do not delete the source object (department). *For example*, if you want to remove a department, you must ensure that all the employees in that department are first transferred elsewhere; otherwise, the department cannot be deleted.
- **Nullify:** Remove the relationship between the objects, but do not delete either object. This only makes sense if the department relationship for an employee is optional, or if you ensure that you set a new department for each of the employees before the next save operation.
- **Cascade:** Delete the objects at the destination of the relationship when you delete the source. *For example*, if you delete a department, fire all the employees in that department at the same time.
- **No Action:** Do nothing to the object at the destination of the relationship. *For example*, if you delete a department, leave all the employees as they are, even if they still believe they belong to that department.

# **What is the best way to do multi threading in Core Data ?**

The best way, according to Marcus Zarra, is not the fastest, but it is by far the easiest and most maintainable. It relies on API’s that Apple introduced with iOS 6, which allows to define child MOCs and also specify a MOC’s concurrency type. The design that Zarra presents is based on how `NSManagedDocument` works and uses:

- A single persistent store coordinator.
- A private MOC which is the only one that actually accesses the PSC.
- A main MOC associated to the UI that is a child of the private MOC.
- Multiple child MOCs that are specific to secondary threads.

![https://miro.medium.com/max/1000/0*KWOPQE-VL9Wyl6Dv.jpg](https://miro.medium.com/max/1000/0*KWOPQE-VL9Wyl6Dv.jpg)

[Source: Infoq](https://www.infoq.com/news/2015/11/zarra-core-data-threading)

The nice thing about this design is that all changes in a child MOC will automatically propagate to its parent MOC, thus doing away with the need for merging.

The **main drawback of this design is its slowness**, although only by a few percents, says Zarra. A tricky issue with it is that if too many async operations are carried through, there may be ripple effects on the UI, since its associated MOC will receive multiple changes in sequence that might not be coherent with one another. An important detail about this design is that it is best not to reuse child MOCs, which are cheap to create. On the other hand, long-lived child MOCs should be manually kept in sync with the main MOC, since changes are propagated only from child MOCs to their parent and not the other way round.

### Ideal Core Data Stack!!!

![https://miro.medium.com/max/566/1*QwNOFMnFzaTdoPtoQyZoxQ.jpeg](https://miro.medium.com/max/566/1*QwNOFMnFzaTdoPtoQyZoxQ.jpeg)

The managed object context linked to the persistent store coordinator isn’t associated with the main thread. Instead, it lives and operates on a background thread. When the private managed object context saves its changes, the write operation is performed on that background thread.

The private managed object context has a child managed object context, which serves as the main managed object context of the application. The concept of parent and child managed object contexts is key in this scenario.

In most scenarios, a managed object context is associated with a persistent store coordinator. When such a managed object context saves its changes, it pushes them to the persistent store coordinator which further pushes the changes to the persistent store, a SQLite database for example.

A child managed object context doesn’t have a reference to a persistent store coordinator. Instead, it keeps a reference to another managed object context, a parent managed object context. When a child managed object context saves its changes, it pushes them to the parent managed object context. In other words, when a child managed object context saves its changes, the persistent store coordinator is unaware of the save operation. It is only when the parent managed object context performs a save operation that the changes are pushed to the persistent store coordinator and subsequently to the persistent store.

Because no write operations are performed when a child managed object context saves its changes, the thread on which the operation is performed isn’t blocked by a write operation. That is why the main managed object context of the application is the child managed object context of a managed object context that operates on a background thread.

참고: [iOS Interview Questions Part 5: Core Data 📗](https://medium.com/@chetan15aga/ios-interview-questions-part-5-core-data-dc97e29835f8)
