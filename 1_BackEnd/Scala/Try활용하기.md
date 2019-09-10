#   Try 활용하기

    import scala.util.{Failure, Success, Try}
    def testTry(param: String) : String = {
        if( param.equals("succ")) "succ"
        else throw new Exception(" failed...");
    }

    val succ = Try(testTry("succ")).toOption match {
                    case Some(x) => x
                    case None => "none"
                }

    val fail = Try(testTry("")).toOption match {
                    case Some(x) => x
                    case None => "none"
                }

    println(succ) // "succ"
    println(fail) // "none"

