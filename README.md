### Небольшой конспектный материал по курсу 
#### (Java from Computer Science Center)

#### Потоки байт. Контрольная сумма прочитанных данных

``` Java
import java.io.*;

public class Main {
    
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
}
```

#### Преобразование переводов строк из формата Windows "/r/n" в формат Unix "/n"

* Input/Output Stream - работают с потоками байтов
* Reader/Writer - работают с потоками символов Unicode

``` Java
import java.io.*;

public class Main {

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
import java.util.Locale;
import java.util.Scanner;

class Main {

  public static void main(String[] args) {
    Scanner sc = new Scanner(System.in).useLocale(Locale.forLanguageTag("ru"));
    double sum = 0;

    while(sc.hasNext()) {
      String tmp = sc.next();
      try {
        sum += Double.parseDouble(tmp);
      } catch (NumberFormatException ignored) { }
    }
    System.out.printf("%.6f", sum);
  }
}

```
