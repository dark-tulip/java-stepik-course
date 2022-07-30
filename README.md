### Небольшой конспектный материал по курсу 
#### (Java from Computer Science Center)

#### Потоки байт. Контрольная сумма прочитанных данных

``` Java
    public static void main(String[] args) throws IOException {
        assert checkSumOfStream(new ByteArrayInputStream(new byte[]{0x33, 0x45, 0x01})) == 71;
    }

    public static int checkSumOfStream(InputStream inputStream) throws IOException {
        byte[] buf = new byte[1];  // block size to read per iteration
        int controlSum = 0;
        while (inputStream.read(buf) > 0) {  // input stream will be written on buf
                controlSum = Integer.rotateLeft(controlSum, 1) ^ (buf[0] & 0xFF);  // cast to unsigned byte
        }
        return controlSum;
    }
```

#### Преобразование переводов строк из формата Windows "/r/n" в формат Unix "/n"

* Input/Output Stream - работают с потоками байтов
* Reader/Writer - работают с потоками символов Unicode

``` Java
static final byte NEW_LINE     = 0x0A;  // '\n' представляется байтом 10, символ
static final byte CARRIAGE_RET = 0x0D;  // '\r' — байтом 13

public static void main(String[] args) throws IOException {
    int prev = System.in.read();  // read one byte
    while (prev > 0) {
        int curr = System.in.read();
        if (prev != CARRIAGE_RET || curr != NEW_LINE) {
            System.out.write(prev);
        }
        prev = curr;
    }
    System.out.flush();
}
```
#### Make string from inputStream
``` Java
public static String readAsString(InputStream inputStream, Charset charset) throws IOException {
    StringBuilder result       = new StringBuilder();
    InputStreamReader isReader = new InputStreamReader(inputStream, charset);
    BufferedReader reader      = new BufferedReader(isReader);
    int ch;
    while ((ch = reader.read()) > 0) {
        result.append((char) ch);
    }
    return result.toString();
}
```
#### Read characted in unsigned bytes
``` Java
byte[] bytes = "Ы".getBytes(StandardCharsets.UTF_8);
for (byte bt : bytes) {
    System.out.println(bt & 0xFF);  // make unsigned
}
```

* PrintWriter не бросает IOException (Оборачивает Writer, устанавливают внутренний флаг ошибки checkError)
* PrintStream не бросает исключений (Оборачивает OutputStream)
* System.out.write -- вывод двоичных данных
* print -- вывод текстовых данных (строка)

#### Сумма чисел из входящего потока байтов
``` Java
Scanner sc = new Scanner(System.in).useLocale(Locale.forLanguageTag("ru"));
double sum = 0;

while(sc.hasNext()) {
    String tmp = sc.next();
        try {
            sum += Double.parseDouble(tmp);
        } catch (NumberFormatException ignored) { }
    }
System.out.printf("%.6f", sum);
```
* java.io (InputStream, OutputStream, Reader, Writer) - блокируется в ожидании пока из потока не будет прочитан хотябы один байт
* java.nio (Channel, ByteBuffer) - низкоуровневый интерфейс неблокирующего ввода вывода, эффективный для масштабирования программ

#### Сериализация и десериализация
- Реализуют интерфейс без методов, где компилятор сам решает как сериализовывать 
- Сериализация - сохранение состояния объекта в байтах 
``` Java
animals.toArray(new Animal[0]);  // cast from list to array
```
``` Java
public static Animal[] deserializeAnimalArray(byte[] data) throws IllegalArgumentException {
    try (ObjectInputStream ois = new ObjectInputStream(new ByteArrayInputStream(data))) {
      Animal[] animals = new Animal[ois.readInt()];
      for(int i = 0; i < animals.length; i++) {
        animals[i] = (Animal) ois.readObject();
      }
      return animals;
    } catch (Exception e) {
      throw new IllegalArgumentException(e);
    }
}
```

#### Дженерики
Параметризация возможна только с ссылочными типами
Один объект производного типа Optional
Optional позволяет писать код без if (ifPresent(), orElse())
<T> - угловые скобки с именами дженерик параметров
Diamond <> оператор работает только с new
Conusmer - принимает один параметр указанного типа (consumer.accept(value))

* final keyword is used in several contexts to define an *entity that can only be assigned once*.
    
``` Java
import java.io.*;

class Pair<T, U> {

    private final T first;
    private final U second;

    private Pair(T obj1, U obj2) {
        this.first = obj1;
        this.second = obj2;
    }

    public static <T, U>Pair<T, U> of(T obj1, U obj2) {
        return new Pair(obj1, obj2);
    }

    public T getFirst() {
        return this.first;
    }
    public U getSecond() {
        return this.second;
    }

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;

        Pair<?, ?> pair = (Pair<?, ?>) o;

        if (first != null ? !first.equals(pair.first) : pair.first != null) return false;
        return second != null ? second.equals(pair.second) : pair.second == null;
    }

    @Override
    public int hashCode() {
        int result = first != null ? first.hashCode() : 0;
        result = 31 * result + (second != null ? second.hashCode() : 0);
        return result;
    }

    public static void main(String[] args) {
        Pair<Integer, String> pair = Pair.of(1, "hello");
        Integer i = pair.getFirst(); // 1
        String s = pair.getSecond(); // "hello"

        Pair<Integer, String> pair2 = Pair.of(1, "hello");
        boolean mustBeTrue = pair.equals(pair2); // true!
        boolean mustAlsoBeTrue = pair.hashCode() == pair2.hashCode(); // true!
    }
}
```
#### Collections
    
``` Java
/* Заполнение множества с условием */
public static <T> Set<T> symmetricDifference(Set<? extends T> set1, Set<? extends T> set2) {
        Set<T> result = new HashSet<>();
        set1.forEach(elem -> { if (!set1.contains(elem)) result.add(elem); });
        set2.forEach(elem -> { if (!set2.contains(elem)) result.add(elem); });
        return result;
}
```

#### Functional Interfaces (from Java 8)
- Ровно один абстрактный метод
- @FunctionalInterface
- any number of default or static methods (fields)
- Java нету указателей на фукнции (как в С, С++ / делегатов как на С#)
- <T> параметризация типом объекта который будет принимать
- Consumer - потребители (принимает но ничего не возращает)
- Supplier - поставщики - не принимают ничего в качестве параметра и что то возращают
- Predicate - принимает значение какого-то типа и возращает boolean
- Function - 
- UnaryOperator - вход и выход одно и то же значение
- Из лямбды нельзя изменять значения переменным содержащим ее методы

1) именованный или анонимный класс
2) лямбда выражения
3) ссылки на метод - только статический

