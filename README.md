**1. Intro**

This is a sample Play App used in the Scala Crash Course Day 4

http://www.meetup.com/%CE%BB-Lambda-ES/events/192225692/

Shows basic usage of a Play2 app with the Play Reactive Mongo Plugin

**2. Adding dependencies**

*/build.sbt*
```scala

resolvers += "Sonatype Snapshots" at "https://oss.sonatype.org/content/repositories/snapshots/"

libraryDependencies ++= Seq(
  "org.reactivemongo" %% "play2-reactivemongo" % "0.10.5.akka23-SNAPSHOT"
)

```

**3. Configuring plugins**

*/conf/play.plugins*

```
1100:play.modules.reactivemongo.ReactiveMongoPlugin
```

**4. Configuring the DB**

*conf/application.conf*
```
mongodb.uri ="mongodb://username:password@localhost:27017/your_db_name"
```

**5. Routes**

*conf/routes*
```routes
GET        /people              controllers.Application.people
POST       /people              controllers.Application.addPerson()
```

**6. Model**

```scala
import play.api.libs.json.Json

case class Address(street : String)

case class Person(name : String, lastName : String, address : Option[Address] = None)

trait JSONFormats {

  implicit val addressFormats = Json.format[Address]
  implicit val personFormats = Json.format[Person]

}
```

**7. Controller**

*controllers/Application.scala*
```scala

object Application extends Controller with MongoController with JSONFormats {

  def collection: JSONCollection = db.collection[JSONCollection]("people")

  def people = Action.async(parse.anyContent) { request =>
    val peopleList = collection.find(Json.obj()).cursor[Person].collect[List](upTo = 100, stopOnError = true)
    peopleList map (list => Ok(Json.toJson(list))) recover {
      case e : Exception => InternalServerError("caput")
    }
  }

  def addPerson = Action.async(parse.json) { request =>
    request.body.validate[Person] match {
      case JsSuccess(person, _) =>
        val insertResult = collection.insert(person)
        insertResult map (_ => Created("to pa dentro"))
      case JsError(error) => Future.successful(BadRequest(error.toString()))
    }

  }

}
  
```

**8. Run this app**

```sh
./activator run
```