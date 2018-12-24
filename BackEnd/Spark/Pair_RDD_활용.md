# Pair RDD 활용

> Pair RDD란? Key와 Value의 형태로 데이터를 저장하는 RDD로 일반적으로 생각하는 Map과 비슷한 데이터 모델이라고 생각하면 된다. 


### pair RDD 생성하기

    var pairRDD = targetRDD.map(tran => (tran(2).toInt, tran))

각각의 row가 String의 배열로 되어있는 RDD의 각 row를 key와 value의 형태로 변형해 새로운 Pair RDD를 만들었다.

+ **Pari RDD Key값 가져오기 및 응용**



        pairRDD.keys.collect
        // 키값 나열하기
        pairRDD.keys.distinct.count
        // 중복값 제거하고 갯수세기
        pairRDD.countbyKey
        res6: scala.collection.Map[Int,Long] = Map(69 -> 7, 88 -> 5, 5 -> 11, 10 -.....
        // 키별로 데이터 개수를 센 후, map의 형태로 리턴
        transByCust.countByKey.values.sum
        // 이 결과는 결국 모든 데이터의 갯수를 세는것과 동일한 결과를 가져온다.
        val(cid, count) = tranbyCust.countByKey.toSeq.sortBy(_._2).last
        // value로 정렬해 가장 마지막 수. 즉, 가장 높은 value를 가진 데이터를 리턴한다. 
        // _._2는 각 데이터의 Value를 의미한다
    

+ **lookup**

        pairRDD.lookup(53)
        res19: Seq[Array[String]] = WrappedArray(Array(2015-03-30, 6:18 AM, 53, 42, 5, 2197.85), Array(2015-03-30, 4:42 AM, 53, 44, 6, 9182.08), Array(2015-03-30, 2:51 AM, 53, 59, 5, 3154.43), Array(2015-03-30, 5:57 PM, 53, 31, 5, 6649.27), Array(2015-03-30, 6:11 AM, 53, 33, 10, 2353.72), Array(2015-03-30, 9:46 PM, 53, 93, 1, 2889.03), Array(2015-03-30, 4:15 PM, 53, 72, 7, 9157.55), Array(2015-03-30, 2:42 PM, 53, 94, 1, 921.65), Array(2015-03-30, 8:30 AM, 53, 38, 5, 4000.92), Array(2015-03-30, 6:06 AM, 53, 12, 6, 2174.02)

key값이 53인 모든 values를 리턴한다.
lookup연산자는 결과값을 드라이버로 전송하므로 이 결과를 메모리에 적재할 수 있는지를 먼저 확인해야 한다. 

        pairRDD.lookup(53).foreach(tran => println(tran.mkString(",")))
        2015-03-30,6:18 AM,53,42,5,2197.85
        2015-03-30,4:42 AM,53,44,6,9182.08
        2015-03-30,2:51 AM,53,59,5,3154.43


+ **mapValues** 

        pairRDD = pairRDD.mapValues(tran => {
            if(tran(3).toInt == 25 && tran(4).toDouble >1)
                tran(5) = (tran(5).toDouble + 0.95).toString
            tran
        })

각각 데이터에서 특정 조건을 만족하는 value를 변형해 새롭게 할당해준다.
이런 작업은 앞서 pairRDD를 var타입으로 지정했기 때문에 가능한 것.


+ **flatMapValues**

각 키값을 0개 또는 한개 이상의 값으로 매핑해 RDD에 포함된 요소 개수를 변경할 수 있다. 즉, 키에 새로운 값을 추가하거나 삭제할 수 있다.

flatMapValues는 변환함수가 반환한 컬렉션의 값들을 원래 키와 합쳐 새로운 key-value쌍으로 생성한다.

        pairRDD = pairRDD.flatMapValue(tran =>{
            if(tran(3).toInt == 81 && tran(4).toDouble >=5){
                val cloned = tran.clone()
                cloned(5) = "0.00";
                cloned(3) = "70";
                cloned(4) = "1;
                List(tran, cloned) // 반환한 컬렉션, 새로운 value가                             추가되었다. 즉, 데이터 갯수가                            늘어남
            } else {
                List(tran) // 기존의 value만 리턴함. 
            }
        }


+ **reduceByKey**

각 키의 모든값을 동일한 타입의 단일값으로 병합

+ **foldByKey** 

reduceByKey와 기능은 같지만, merge함수의 인자 목록 바로 앞에 zeroValue인자를 담은 또 다른 인자 목록을 추가로 전달해야 한다. 

    > foldByKey(zeloValue: V)(func :(V,V) => V)

zeroValue는 반드시 항등원이어야 한다. 덧셈에서는 0, 곱셈 혹은 나눗셈에서는 1 등이 바로 항등원이다.

    val amounts = pairRDD.mapValues( tran => tran(5).toInt)
    val total = amounts.foldbyKey(0)((a,b)=> a+b)
    total.collect()
    Array[(Int, Double)] = Array((84,53020.619999999995), (39,49842.78), ...

+ **aggregateByKey** 

zeroValue와 함께 변환함수 및 병합함수를 인자로 받는다. 
즉 변환함수는 각 파티션별로 작업을 진행하며, 그렇게 얻어진 결과를 두번째 함수인 병합함수에서 합산하게 된다. 

    val prods = pairRDD.aggregateBykey(List[String]())(
        (prods, tran) => prods:::List(tran(3)), // 각각의 파티션에                                         // 대해 수행
        (prod1, prod2) => prod1:::prod2 // 파티션별로 넘겨받은         
         // prod에 대해 수행해 하나의 배열을 리턴한다. 
    )
> :::는 두개의 배열을 이어붙이는 연산자이다.



