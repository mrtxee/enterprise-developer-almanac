---
aliases:
  - javap
  - class
  - .class
---
# Версия .class vs версия Java

При компиляции Java-файла создается файл _.class_. В некоторых случаях нам нужно узнать версию Java, в которой был скомпилирован файл класса. **Каждой основной версии Java соответствует основная версия файла _.class_, который она генерирует.**

В этой таблице мы сопоставляем основной номер версии _.class_ с версией JDK, в которой была представлена эта версия класса, и показываем шестнадцатеричное представление этого номера версии:

|Выпуск Java|Класс Основной версии|Шестнадцатеричный|
|---|---|---|
|Java SE 18|62|003e|
|Java SE 17|61|003d|
|Java SE 16|60|003с|
|Java SE 15|59|003b|
|Java SE 14|58|003a|
|Java SE 13|57|0039|
|Java SE 12|56|0038|
|Java SE 11|55|0037|
|Java SE 10|54|0036|
|Java SE 9|53|0035|
|Java SE 8|52|0034|
|Java SE 7|51|0033|
|Java SE 6|50|0032|
|Java SE 5|49|0031|
|JDK 1.4|48|0030|
|JDK 1.3|47|002f|
|JDK 1.2|46|002e|
|JDK 1.1|45|002d|


`javap` — это утилита командной строки из [Java Development Kit](https://www.google.com/search?q=Java+Development+Kit&mstk=AUtExfATrImL7FAC1a9rTmmGS2Z-1J6IzcMW2kokYLFNMoInrwymVmC44N_I6_XVE3t1UXGyspQvMTZaNTxuZv9i05XOs8e6IGP2YoaIxSMgkU-ugiGWX1OaaJtc8vOz2EsgrLDL7VxnlIFkNAErC_dk8-OOwUVZ_gmKJjXVxboCxHOCrt5CFhRyP63XlElN-Wk2xZHpDcPsP5r1Aa7aIdHkt7UQDn9pztko4av_nAY3S-EDNij0HLyRIDiaoF7BUpnvedn7OtdoerKACikjFvA3JSkWaaIsM84cl8P_6Au4GlHvnA&csui=3&ved=2ahUKEwiJ9eHQ8oSQAxWtA9sEHWQyAOEQgK4QegQIARAB) (JDK), которая **дизассемблирует файлы классов Java** (.class), позволяя просмотреть содержащийся в них байт-код и сигнатуры методов. Она анализирует структуру файла класса и выводит информацию о полях и методах класса, а также байт-код JVM, что помогает понять, как работает конкретный оператор или увидеть эффект от опций компилятора.