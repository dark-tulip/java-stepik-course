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



