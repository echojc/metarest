MetaRest
--------
Use Scala macros to generate your RESTy models

Let's say you have the following `User` model in your business layer:

```scala
case class User(id: Int, name: String, email: String, registeredOn: DateTime)
```

But, now you want to create well-formed models to describe the requests/responses of your HTTP REST APIs:
```scala
// Response to GET /users/$id (Retrieve an existing user)
case class UserGet(id: Int, name: String, email: String)

// Request body of POST /users (Create a new user)
case class UserPost(name: String, email: String)

//Request body of PATCH /users/$id (Edit name of an existing user)
case class UserPatch(name: Option[String])
```

That is a lot of boilerplate! Keeping all these request models in sync with your business model and/or adding/removing fields quickly becomes difficult and cumbersome for more complicated models.

With MetaRest, all you need to do is:
```scala
import com.github.pathikrit.MetaRest, MetaRest._

@MetaRest case class User(
  @get               id            : Int,
  @get @post @patch  name          : String,
  @get @post         email         : String,
                     registeredOn  : DateTime
)
```

The above block would generate code essentially looking like this:
```scala
object User {
  case class Get(id: Int, name: String, email: String)
  case class Post(name: String, email: String)
  case class Patch(name: Option[String])
}
```

Now, you can have a well defined CRUD interface:
```scala
trait UserRepo {
  def get(id: Int): User.Get
  def create(request: User.Post): User.Get
  def update(id: Int, request: User.Patch): User.Get
}
```

MetaRest also automatically generates [spray-json](https://github.com/spray/spray-json)'s formatters for all the models using the [spray-json-annotation](https://github.com/ExNexu/spray-json-annotation) macro:

```scala
import spray.json._
import DefaultJsonProtocol._

val jsonStr: String = """{
  "name": "Rick",
  "email": "awesome@msn.com"
}"""
val request: User.Post = jsonStr.parseJson.convertTo[User.Post]
val json: JsValue = request.toJson

println(s"REQUEST=$request", s"JSON=$json")
assert(json.prettyPrint == jsonStr)
```
