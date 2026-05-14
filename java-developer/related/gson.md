Object o → JSON → Object o;  
библиотека Google  
[Gson](https://code.google.com/p/google-gson/) предназначена для преобразования Java-объектов в текстовый формат `JSON (сериализация)` и обратного преобразования (`десереализация`).
[[#dependency]]
[[#toJson() – Java object to JSON]]
[[#Prettyprint]]
[[#fromJson() - get object from JSON]]
[[#class TypeToken]]
# dependency
`import` **`com.google.code.gson;`**
```XML
<!-- https://mvnrepository.com/artifact/com.google.code.gson/gson -->
<dependency>
  <groupId>com.google.code.gson</groupId>
  <artifactId>gson</artifactId>
  <version>2.10.1</version>
</dependency>
```
[https://mkyong.com/java/how-do-convert-java-object-to-from-json-format-gson-api/](https://mkyong.com/java/how-do-convert-java-object-to-from-json-format-gson-api/)
# toJson() – Java object to JSON
```Java
Gson gson = new Gson();
String json = gson.toJson(obj);
System.out.println(json);
String fileName = System.getProperty("user.dir")+"\\emailValidatorFSM.json";
FileWriter fw = null;
try {
    fw = new FileWriter(fileName);
    BufferedWriter bw = new BufferedWriter(fw);
    gson.toJson(obj, bw);
    bw.flush();
} catch (IOException e) {
    e.printStackTrace();
} finally {
    try {
        if (fw != null) {
            fw.close();
        }
    } catch (IOException e) {
        e.printStackTrace();
    }
}
```
## Prettyprint
```Java
// pretty print
Gson gson = new GsonBuilder().setPrettyPrinting().create();
String json = gson.toJson(obj);
```
# fromJson() - get object from JSON
```Java
//easy
StateTransitionSet obj =  (new Gson()).fromJson(new FileReader(fileName), StateTransitionSet.class);
//complecated
Gson gson = new Gson();
Type listType = new TypeToken<State[]>() {}.getType();
State[] obj =  gson.fromJson(new FileReader(fileName), listType);
```
# class TypeToken
`import com.google.gson.reflect.TypeToken;`
Калсс который входит в библиотеку gson и служит для десерилизации Json-текста в объект java
```Java
Type type = new TypeToken<HashMap<LEXEM, String>>() {
}.getType();
HashMap<LEXEM, String> (new Gson()).fromJson(new FileReader(CONFIG_FILEPATH), type);
```