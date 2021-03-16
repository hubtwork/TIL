### RestTemplate 배열 형식의 Json Mapping

> ✔ **IDE : IntelliJ Ultimate 2020.3.2**
>
> ✔ **Lang: Kotlin**



##### exchange 이용해 배열 형식의 Body 를 CustomObject 매핑

~~~kotlin
// ParameterizedTypeReference :: Array 형태의 JSON Object 매핑
inline fun <reified T> typeReference() = object : ParameterizedTypeReference<T>() {}

val resultEntity: ResponseEntity<ArrayList<ResultClass>> = restTemplate.exchange("uri", HttpMethod.GET, null, typeReference<ArrayList<ResultClass>>())
val resultList: ArrayList<ResultClass>? = resultEntity.body
~~~

