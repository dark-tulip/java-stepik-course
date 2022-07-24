### Базовый курс по java from Computer Science Center


#### Потоки байт. Контрольная сумма прочитанных данных

``` Java
import java.io.*;

public class Main {
    
    public static void main(String[] args) throws IOException {
        assert  checkSumOfStream(new ByteArrayInputStream(new byte[]{0x33, 0x45, 0x01})) == 71;
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
``` Java
import java.io.*;

public class Main {

    public static void main(String[] args) throws IOException {

        byte NEW_LINE     = 0x0A;  // '\n' представляется байтом 10
        byte CARRIAGE_RET = 0x0D;  // '\r' — байтом 13
        byte[] buf = new byte[1];

        if (System.in.read(buf) > 0) {
            byte prev = buf[0];
            while (System.in.read(buf) > 0) {
                byte curr = buf[0];
                if (prev != CARRIAGE_RET || curr != NEW_LINE) {
                    System.out.write(prev);
                }
                prev = curr;
            }
            System.out.write(prev);
            if (prev != NEW_LINE) {
                System.out.write(NEW_LINE);
            }
        };
        System.out.flush();
    }
}
```
