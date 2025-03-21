---
icon: material/text-box-outline
---

# Mobile Analysis

**Difficulty**: :fontawesome-solid-star::fontawesome-solid-star::fontawesome-regular-star::fontawesome-regular-star::fontawesome-regular-star:<br/>

## Objective

!!! question "Request"
    Help find who has been left out of the naughty AND nice list this Christmas. Please speak with Eve Snowshoes for more information.

??? quote "Eve Snowshoes"
    Hi there, tech saviour! Eve Snowshoes and Team Alabaster in need of assistance.

    I've been busy creating and testing a modern solution to Santa’s Naughty-Nice List, and I even built an Android app to streamline things for Alabaster’s team.

    But here’s my tiny reindeer-sized problem: I made a debug version and a release version of the app.

    I accidentally left out a child's name on each version, but for the life of me, I can't remember who!

    Could you start with the debug version first, figure out which child’s name isn’t shown in the list within the app, then we can move on to release? I’d be eternally grateful!

## Hints

??? tip "Missing"
    Maybe look for what names are included and work back from that?

??? tip "Tools"
    Try using apktool or jadx

??? tip "Hard - Format"
    So yeah, have you heard about this new Android app format? Want to convert it to an APK file?

??? tip "Hard - Encryption and Obfuscation"
    Obfuscated and encrypted? Hmph. Shame you can't just run strings on the file.

## Solution

### Diagrams

```mermaid
sequenceDiagram
  autonumber
  Decompile APK->>read: to get smali files from the debug file
  loop Read code files
      read->>read: Search smali files for anything interesting
  end
  Note right of Decompile APK: For example, use APKLab in VSCode
  read->>list: Find the names (and locations) in DatabaseHelper.smali
  apk->>android: Run the APK in a simulated device through Android Studio
  android<<->>list: Compare the displayed list with the smali file
```

!!! success "Ellie"
    Ellie from Alabama, USA is missing

### Gold

Gold is a bit more complex. First, we have to convert the 
[aab into apk](https://github.com/HackJJ/apk-sherlock/blob/main/aab2apk.md).

The information we want is still in DatabaseHelper.java but this time it is obsfucated:
```java
super(context, DATABASE_NAME, (SQLiteDatabase.CursorFactory) null, 1);
Intrinsics.checkNotNullParameter(context, "context");
String string = context.getString(C0793R.string.f175ek);
Intrinsics.checkNotNullExpressionValue(string, "getString(...)");
String obj = StringsKt.trim((CharSequence) string).toString();
String string2 = context.getString(C0793R.string.f176iv);
Intrinsics.checkNotNullExpressionValue(string2, "getString(...)");
String obj2 = StringsKt.trim((CharSequence) string2).toString();
byte[] decode = Base64.decode(obj, 0);
Intrinsics.checkNotNullExpressionValue(decode, "decode(...)");
this.encryptionKey = decode;
byte[] decode2 = Base64.decode(obj2, 0);
Intrinsics.checkNotNullExpressionValue(decode2, "decode(...)");
this.f174iv = decode2;
this.secretKeySpec = new SecretKeySpec(decode, "AES");
```

We can see how the encryption works in another file (C0793R.java):
```java
    /* renamed from: ek */
    public static int f175ek = 2131296307;
    /* renamed from: iv */
    public static int f176iv = 2131296311;
    // Which in res/values/strings.xml shows:
    // Ek: rmDJ1wJ7ZtKy3lkLs6X9bZ2Jvpt6jL6YWiDsXtgjkXw=
    // IV: Q2hlY2tNYXRlcml4
    private final String decryptData(String str) {
        try {
            Cipher cipher = Cipher.getInstance("AES/GCM/NoPadding");
            cipher.init(2, this.secretKeySpec, new GCMParameterSpec(128, this.f174iv));
            byte[] doFinal = cipher.doFinal(Base64.decode(str, 0));
            Intrinsics.checkNotNull(doFinal);
            return new String(doFinal, Charsets.UTF_8);
        } catch (Exception e) {
            Log.e("DatabaseHelper", "Decryption failed: " + e.getMessage());
            return null;
        }
    }
```

We can create our own Java file to decrypt it:

```java
import javax.crypto.Cipher;
import javax.crypto.spec.GCMParameterSpec;
import javax.crypto.spec.SecretKeySpec;
import java.util.Base64;
import java.nio.charset.StandardCharsets;
import java.util.Iterator;

public class DecompileClass {
    public static String decryptData(String str) {
        byte[] encryptionKey = Base64.getDecoder().decode("rmDJ1wJ7ZtKy3lkLs6X9bZ2Jvpt6jL6YWiDsXtgjkXw=");
        byte[] f174iv = Base64.getDecoder().decode("Q2hlY2tNYXRlcml4");
        SecretKeySpec secretKeySpec = new SecretKeySpec(encryptionKey, "AES");
        try {
            Cipher cipher = Cipher.getInstance("AES/GCM/NoPadding");
            cipher.init(2, secretKeySpec, new GCMParameterSpec(128, f174iv));
            byte[] doFinal = cipher.doFinal(Base64.getDecoder().decode(str));
            return new String(doFinal, StandardCharsets.UTF_8);
        } catch (Exception e) {
            //Log.e("DatabaseHelper", "Decryption failed: " + e.getMessage());
            System.err.println("Decryption failed: " + e.getMessage());
            return null;
        }
    }
   
    public static void main(String args[]) {
        String[] names = new String[]{"L2HD1a45w7EtSN41J7kx/hRgPwR8lDBg9qUicgz1qhRgSg==", "IWna1u1qu/4LUNVrbpd8riZ+w9oZNN1sPRS2ujQpMqAAt114Yw==", "MWfO0+M1t5IvQtN2ad9w3hp81sYQIIaX6veq03bnk6I4H/1n89gW", "LmHJ164506skXdh3K9MZ/BBiw90TRO2mD0Hp9Nuoxu4ghx5/WQ==", "J2XF2645xKciX9RgK9MR+wZ60NIbKIsOTRSHP0jkBJPaF0djlqbc", "LGfJ0q451qslWt14aZd8rjtr1ZMtJItIvrKk8RRQWh2U6bQSEdPga59XDQ==", "LWTBzOt4u/4KXt99aJ18riBgy8cSJcpvtrKnM4IEsMDr9AwtlSJW+S/jdoHvXA==", "I2HM3+w1t4opQ953c5x8rjZvzNITIEFFaMC3bP4bW/FptwSEIzo=", "K3vJ2Od1+79qEeN2apZ8rjx6w98OKXRQNvvj/tYXj2mFoDhaZw==", "O33D06452K0nWtA1J7kx/hRgtZNzI71BDpxorJ6mxImw/A==", "L2na0+M1t58yWdR3dN9wyQdrx9ASZFBP13TAtgvjKYVFW9J6rA==", "KmnGya451bs0Xdh3K9MX6wdjw90OdugnqIwKHDmVzoF7PjkQJQ==", "IWDN1K451bsvW9h3YN9wzR1nzNLx7AaHK857pwQ3GlLQ2i+e", "I2XByK451L8vQ941J7Y39wV6pk1FpbJdc3QZRlpVuVIsjQ==", "JGnc0+94u/4FUMJ4ZZ8x4BZvjpM6LphLvL+vfIsRMj38FLu+GyCyUdPJ2g==", "I2TNwq452bsxEeh2dZh8riBd4zTtytJ89Bxu9NEok+aDWmI=", "J2TN1OM1t5MpQtJ2cN9w3AB90doWBr/HMmnvw088K+A8OYyeqg==", "Ln3L26452rcqUN81J7ok7xl32UJTvrQCKVSd6Z+tiX2Hmg==", "Lm3B1uM1t5wjWMNsc99wwhBsw90YLyjPYh2rLfMJ1C4VX2kQBpE=", "KGfG2/E1t40yXtJyb5w841ku8cQSJY9K1bwgTiInx6cYV5Ui/DM+sA==", "L23BlqJK/78oVtl4bt9wzR1nzNKBr23V1KlRgUeyT5PagT9r", "MWnFz+d1u/4EVMN3K9MD+Rx62NYFLYtKu34IoIzUoRVftPjhSrdCxro=", "LGnM0+M1t5UvVMc1J6Y7/BRnzNY2Zug0wYGSAMBLA1AcU9yx", "NGHLzu1ru/4ERNJxZoE1/QEiguEYLItKtr0WsmwcMfhswLv9Tec+ZRjR", "I2Hb0uM1t5AnWMN2ZZp8rj5rzMoWa/+6xCGsNnHvb00IU4Fr3Q==", "JmnG0+d1u/4JQt12K9Me4Qd5w8o6jn3t2IdQwSuXYx7u//Cv", "I2bBzuM1t5MzXNN4bt9wxxtqy9JRetksUopAr2UrK5xAUHDW", "JmHN3e01t5MjSdh6aNMT5wF3jpM6JJJNvLOoRJgv4sLKLQau2YBLzokQ", "LmHG3uM1t50pQdR3b5I36xsigvcSL4dFrbfU9Cqvl5I/xLcz0XU03pLK", "I2bc1ew1t4gvVN93Zt9wzwB91sEeIIGZ9QAB6FA50LlDQiHup1s=", "MWna264537sqQth3bJp8rjNnzN8WL47gYBHKOnMA/05AZVXhUewu", "KGfa3ec1t5IvXNA1J6M1/ACz8Z0iiXnvwFsEg5jrkLGq", "JXrJ2ec1t5siWN97coE35lku8dAYNYZFsbi05FzIyksNODX0NjFqyvVn", "L2nc3+01t5wzVN92dNMR5wdr0Z9XAJhDurK0PoMIwB3/YInPpneEf/Q3blsrDg==", "J2XB1vs1t5ozU91wad9wxwdrztIZJQMfNE4DYwx1R7bg5c50NDQ=", "Jmne0+Y1t5QpWdB3aZYj7AB8xZ9XEoVRq7TgFosbO6NUOYiKnRWuNaKuxMtDzqVA7Q==", "LGfa26451rM1RdRrY5I9olVAx8cfJJhIvrKkJJm3opYYxGiM9SQhFp8qlek=", "MXzN3ON3u/4EVN1+dZI061ku8dYFI4NF6DAFvK8r77cS9dUOpitQRg==", "KWHFlqJK8rEzXZ05VJwl+h0u6dwFJIuDD8ONR42MoY7KFht/QJfy", "Ln3L0+M1t5wnQ9J8a5w+71ku8cMWKIQC0zxiXHG/YZvw7T+TNIqz", "LWXJyK451rMrUN81J7k//BFvzGO14j2tV1Au4CjIir5oH4s=", "L2na0+c1t44nQ9hqK9MW/BRgwdazin6FJqgBFVKkiYZE/Oku", "NmfFlqJa/7clUNZ2K9MF3TR3qVJdK8qDeVx+ZJ0MHnNU", "Km3E3+x4u/4WQ9B+cpZ8rjZ0x9AfYbhBr6miO4QKWNrxWDpCUMmjm7xYtCdc6A==", "I3rCz+w1t5ojXdlwK9MZ4BFnwzwsgLd0z5uSRxTqIB8aQs4=", "L2nazuM1t4knQ8J4cN9w3hpiw90TTjWQOk7MWqw/XG0TO0AeQA==", "MWHF1ew1t4QzQ9h6b99w3QJn1skSM4ZFsbgRVLExuZmptZSHE6Yrul/e", "I2bJlqJD9rk0VNM1J7Ai4RR6y9JIjLKEIfFIhBPBpLBEqomH", "MGnO2+d1u/4KWMJ7aJ18riVh0McCJotIhSM4dPg694vcV/24WexvGQ==", "LmHG3uM1t4wjSNpzZoU55Vku69ASLYtKu2L+bDaBlZjDIaGNeZhBJqA=", "L2HL0uN1u/4EQ9BtboA87wNvjpMkLYVSvrepNgEBI1AclZNI8gyCBYQG2eM=", "J37JlqJb4ronQdRqc99wxgBgxdIFOAlaPoAgk8c4iPDAzcEuNDE=", "LGHD1e54/vJmfNh3dJh8rjdrztIFNJmu2xg+J6QSEiIx6k9QIl+a", "KWDJ3utz9vJmddByZoF8riZrzNYQIIbM23+WAVV6B504yBF/YTdb", "O2fdyfF88fJmY9B7Zod8rjhh0NwUIoVxNH9I0bHcUbdrTrTpnIWL", "LmndyOM1t5IvXNA1J6M1/ADJIEAtaTSWOs3rUHGxE23v", "KH3J1K45xqsvRd41J7Yz+xRqzcGKc2ANFJm7yAP/iEAOkACn", "I2TBlqJN4rAvQp05U4Y+5wZnw7REbFTEsKKaYtEbT4D/BfA=", "I2TB1OM1t40pV9h4K9MS+xlpw8EeIHcUEuddwvPOGYIPlGbXuns=", "JmHF0/Zr/vJmZdNwa5oj51ku5dYYM41Nvhg6ySTDiTI5vv18a6J6y14=", "J2XF2+xs8rJqEf14YJwjolVAy9QSM4NFBawvI9LTb3YoXPhmLC/pDw==", "KH3E0+M1t4gvVN93Zt9wzwB91sEeIEtgVtuakzge6p44dqbJkEc=", "MGfK3/Btu/4BXdBqYJwnolVdwdwDLYtKu+LDoHra5pWd9cdD5y0f3Mo=", "I2bJyfZ45LcnHZFKZpo++lVex8cSM5lGqq6ne807J7NGCggtcc93Ohi+VKRgXDKpmdgI", "J2bS1a452r80QtRwa581olVI0NIZIo+b1vqa1HqNcwPKRRUCkA2e", "I2Hb0uM1t5QnWtBrc5J8rjxgxtwZJJlNvvHYzXGTBJ/ect8fCv2lmE4=", "L2nc3/dq7fJmesN4bJwnolVezd8WL47o08C8QCvrmNi28s6uzbkB", "MGfb2645378wUN94K9MT+xdvICDOUBRXAjmOpELQySokFQ==", "JmHN3e01t40nX8V2J7c/4xxgxdxbYa5LsrWuPo4IPOBnBhkbfJx6kuAQ5S2AHRJNPm+/JMsu2Y4=", "K3vJ2Od1u/4LUN9wa5J8riVmy98eMZpNsbmz/o7XqAaLAwARWkH6eIgkvg==", "JmnG0+d1u/4FUMN4ZJIjolVYx90SO59Bs72pxXLFYIQzfSavfBYeKvSp", "MWfO0+M1t40pV9h4K9MS+xlpw8EeIOWh30NfCMogzRbG9bbOpOQ=", "L33bzuN/9vJmcN9yZoExolVa18EcJJMyCPdCGbm3ITUv4dWqaqTJ", "JW3HyOV8u/4IUNhraJE5olVFx90OIIDivPFcm6m1iBokvOvJ6Jw=", "LmHG26451L8vQ941J7Y39wV6CD4JBxN6y16SruzezWKPkA==", "KGfb3645xL8oEft2dDD5olVNzcADIMp2tr+hjSygfMWJwjL2dFscDmwjjg==", "L2HJlqJN9rIqWN93K9MV/QFhzNoWqj9VHhFMZ8Go9BUGzcsDyA==", "J2TB2/E1t5YjXcJwaZg5olVIy90bIIRA/1z33GgMGZTTRJFra6XBzA==", "OGna26453q0qUNx4ZZI0olVew9geMp5FsXmupyWxdH/oy9rxaRJqUOA=", "I2zJ16451awnRdhqa5Im71ku8d8YN4tPtr1l2Twrd2VbP7DifnHx1H8R", "J2TB3K453q0yUN97cp98riF70NgSOCtbDxARU2xtlLgyGO7yaRU=", "Ln3LlqJb5as1QtR1dN9wzBBixdoCLK9D8qtP2JBdQm86rVaciwM=", "LmndyOM1t5MnVcNwY99w3QVvy916DDmwWl6gWeYjk2X3jVl2", "NGHLzu1ru/4ERNJxZoE1/QEiguEYLItKtr0WsmwcMfhswLv9Tec+ZRjR", "I2TNwuN386wjHZFVcos14xdh18EQbcpoqqSlOo8GJ7JSkODr7+fBPtiGNds2i5nCLw==", "KGfb0vd4u/4EWMN0bp035hRjjpMiL4NQurjgHIQHNaRaDnIYbKQ9JusGaa1aAkGEVV8=", "J2XF26451qslWt14aZd8rjtr1ZMtJItIvrKk5S9JSEZ8qKvgRP2Mvcy4eQ==", "Ln3Bya45xL8oRdh4YJx8rjZmy98Sl+3RcG/g8aeHWMyfJPImOg==", "IWnc0udr/rAjHZFWc4cx+RQigvAWL4tAvg5ALUCnxBG/cw3LfEmj4oA=", "NmfFeSNqu/4LXt9tYoU56hBhjpMiM59Dqr253YTDn38uXKJ5tl00FEINTg==", "I2bG26451LE2VN9xZpQ14Fku5tYZLItWtJEJuhXmygDg52L+RBvqJyA=", "Mm3c3/A1t5E1Xd41J70//AJv27+zexRjk1E0oOdUtHEChqA=", "KH3E0+M1t5wjQ91wad9wyRB8z9IZOJNUHr+RrQebnoaStFwY+6Y=", "MWnFz+d1u/4QWNR3aZJ8rjR70ccFKItX81FQp//+9EtF+nURL4FI", "LGHG264527QzU91zZp0xolVdztwBJIRNvlt7/VakIoRl++rXlHttwhE=", "I2bMyOdwu/4FWdhqbp0x+1ku79wbJYVSvr7qGO5Zb23L3xCvXHMRYvs=", "KWnBlqJR8rI1WN9ybt9wyBxgztIZJUkQdjl75fDWiRSzBI1CHdw=", "I2DF3+Y1t50nWMN2K9MV6Qx+1mbIOp5nPzrOusVi9a3n/50=", "O2nb1+t38vJmctBqZpE87xttw59XDIVWsL+jOGYAZSakGQcwH9LkY0MNP74=", "Lm3H1K45zb8hQ9R7K9MT/Bpv1toWCeg3YKGO+eApkQGBF09JMw==", "K3vJ2Od1+79qEeN2apZ8rjx6w98OKXRQNvvj/tYXj2mFoDhaZw==", "JG3E0/o1t5MzX9h6b99wyRB8z9IZOEuf/8PQJ7tJYwF2ZBYDZDY=", "L2nc3+01t5wpVt5txFJ8rjZhztwaI4NFdKbrVcU/JGUSzKJm3q3PSQ==", "LWTBzOt4u/4RVN11bp03+hpgjpM5JJ0EhbmhO4wHNjmjzqwzDO366z8XlmAxBAo=", "JmnG0+d1u/4MXtl4aZ01/Rd70NRbYblLqqiod6wPIKlWAvC4AsA71TXm3aAJve9HIUA=", "LGfa2645278hXsI1J7056RB8y9LIAUNdOdmXkW3fRpcNUxV8", "MXzN3ON3u/4EVN1+dZI061ku8dYFI4NF6DAFvK8r77cS9dUOpitQRg==", "Lm3B1uM1t58qVth8dYB8rjRixdYFKItGdCiZJowXSEk2P7+XCerx", "KWHFlqJb4q0nX505VJwl+h0u6dwFJIuiXi2lY6YPZ4CjViAKRpaC", "Ln3L0+M1t5wnQ9J8a5w+71ku8cMWKIQC0zxiXHG/YZvw7T+TNIqz", "LWXJyK4507EuUJ05VpIk7wcjUnL2xhkJz8n9Dm47dXGt", "L2na0+c1t5IzSdR0ZZwl/BIigv8COY9JvbO1JYqohuoT5OQ0dvvUWILo1jaO", "NmfFlqJK9rBmd8N4aZA5/RZhjpMiEqvWVxuq2sU0ngeuJh5Bmivp", "Km3E3+x4u/4QWNR3aZJ8rjR70ccFKIsDa5wYsSNXEVH6T9VNnoNh", "I3rCz+w1t5wnX9Z4a5wi61ku690TKIvYZgzmW1HQgG0OvS+GZrqX", "L2nazuM1t440UNZsYt9wzQ9rwdtXE49Uqr6sPo4tNlgEErLmy2e9McwjexaZ", "MWHF1ew1t5kjX9RvZt9w3QJn1skSM4ZFsbjUNmfsNMnhkq0GBwNdRuQQ", "I2bJlqJD9rk0VNM1J7Ai4RR6y9JIjLKEIfFIhBPBpLBEqomH", "MGnO2+d1u/4WXsNtaN9w3hp81sYQIIbYlRYHiSlXMoS4tPdilxhG", "LmHG3uM1t4wjSNpzZoU55Vku69ASLYtKu2L+bDaBlZjDIaGNeZhBJqA=", "L2HL0uN1u/4EQ9BtboA87wNvjpMkLYVSvrepNgEBI1AclZNI8gyCBYQG2eM=", "J37JlqJb4ronQdRqc99wxgBgxdIFOAlaPoAgk8c4iPDAzcEuNDE=", "LGHD1e54/vJmfN5qZJwnolVc18AEKIvjrzloQ+cC1iWupZVahPs2", "KWDJ3utz9vJmddByZoF8riZrzNYQIIbM23+WAVV6B504yBF/YTdb", "O2fdyfF88fJmY9B7Zod8rjhh0NwUIoVxNH9I0bHcUbdrTrTpnIWL", "LmndyOM1t5IvXNA1J6M1/ADJIEAtaTSWOs3rUHGxE23v", "KH3J1K45xqsvRd41J7Yz+xRqzcGKc2ANFJm7yAP/iEAOkACn", "L2na0+M1t40nX5FTcpI+olVe19YFNYUEjbWjOJpcl9JvvyJ7scI8el5JTy4=", "I2TBlqJN4rAvQp05U4Y+5wZnw7REbFTEsKKaYtEbT4D/BfA=", "I2TB1OM1t5wzUtl4dZYj+lku8NwaIIRNvvtC1rkdlxBlbKrQAX6AucI=", "JmHF0/Zr/vJmeth8cd9w2x58w9oZJM51yxayaVH4ddqhS9Ld+lI=", "J2XF2+xs8rJqEf14YJwjolVAy9QSM4NFBawvI9LTb3YoXPhmLC/pDw==", "KH3E0+M1t4gvVN93Zt9wzwB91sEeIEtgVtuakzge6p44dqbJkEc=", "MGfK3/Btu/4BXdBqYJwnolVdwdwDLYtKu+LDoHra5pWd9cdD5y0f3Mo=", "I2bJyfZ45LcnHZFKZpo++lVex8cSM5lGqq6ne807J7NGCggtcc93Ohi+VKRgXDKpmdgI", "J2bS1a452rcqUN81J7ok7xl3dnBUBK/NLqVw5yX69lhDyQ==", "I2Hb0uM1t5QnWtBrc5J8rjxgxtwZJJlNvvHYzXGTBJ/ect8fCv2lmE4=", "L2nc3/dq7fJmdtV4aYA7olVezd8WL47176gxNd+vY9fx4MSmT2ej", "MGfb2645378wUN94K9MT+xdvICDOUBRXAjmOpELQySokFQ==", "JmHN3e01t40nX8V2J7c/4xxgxdxbYa5LsrWuPo4IPOBnBhkbfJx6kuAQ5S2AHRJNPm+/JMsu2Y4=", "K3vJ2Od1u/4LUN9wa5J8riVmy98eMZpNsbmz/o7XqAaLAwARWkH6eIgkvg==", "JmnG0+d1u/4FUMN4ZJIjolVYx90SO59Bs72pxXLFYIQzfSavfBYeKvSp", "MWfO0+M1t40pV9h4K9MS+xlpw8EeIOWh30NfCMogzRbG9bbOpOQ=", "L33bzuN/9vJmc9B+b5cx6lku68EWMIJsYV5vA84d075o0zjkKww=", "L23BlqJN9rc2VNg1J6cx5wJvzCDEGFoHhNAE0ZQMzBepH+M=", "JW3HyOV8u/4IUNhraJE5olVFx90OIIDivPFcm6m1iBokvOvJ6Jw=", "LmHG26451bsvQ8RtK9Mc6xdvzNwZZ/8B9lM2+mUZn77BIM0HBQ==", "KGfb3645xL8oEft2dDD5olVNzcADIMp2tr+hjSygfMWJwjL2dFscDmwjjg==", "L2HJlqJN9rIqWN93K9MV/QFhzNoWqj9VHhFMZ8Go9BUGzcsDyA==", "KGfG2/E1t4gvXd9wcoB8rjln1tsCIIRNvmU7T9+1tpsYdtrvmWv1l7U=", "J2TB2/E1t5YjXcJwaZg5olVIy90bIIRA/1z33GgMGZTTRJFra6XBzA==", "OGna26453q0qUNx4ZZI0olVew9geMp5FsXmupyWxdH/oy9rxaRJqUOA=", "I2zJ16451awnRdhqa5Im71ku8d8YN4tPtr1l2Twrd2VbP7DifnHx1H8R", "J2TB3K451rAtUMN4K9ME+wdlx8rZbZqbfhGZr3wzxCxB4bYI", "Ln3LlqJJ9qwvQp05QYEx4BZrln569Gq0kHSy+WS+n1k/sA==", "LmndyOM1t5MnVcNwY99w3QVvy916DDmwWl6gWeYjk2X3jVl2", "NGHLzu1ru/4ERNJxZoE1/QEiguEYLItKtr0WsmwcMfhswLv9Tec+ZRjR", "MWna2645xKopUtpxaJ89olVd1dYTJISsz5CtbxJFXJPiMCte0noG", "I2TNwuN386wjHZFbdYYj/RBi0Z9XA49IuLW1OoUxRdZ+wmfnJvvtJwXi10k=", "L2na0+M1t5IvQtN2ad9w3hp81sYQIIYOpc2XzyjeMgAWf7nbp/ts", "L2HL0uN8+/Jmfd53Y5w+olVbzNoDJI4ElLWuMIkGP9LTeVuwVKu4l7DOB1U91N0=", "J2XB1vs1t5ozU91wad9wxwdrztIZJQMfNE4DYwx1R7bg5c50NDQ=", "Jmne0+Y1t40/Vd98ft9wzwB91sEWLYNFu2xWVc39+4i1Bn7JWwoynA==", "J2XF26451qslWt14aZd8rjtr1ZMtJItIvrKk5S9JSEZ8qKvgRP2Mvcy4eQ==", "Ln3Bya452rs+WNJ2J7A5+gwigv4SOYNHsHi4IVfVImdVO9OdwBX0/mw=", "IWnc0udr/rAjHZFWc4cx+RQigvAWL4tAvg5ALUCnxBG/cw3LfEmj4oA=", "NmfFeSNqu/4LXt9tYoU56hBhjpMiM59Dqr253YTDn38uXKJ5tl00FEINTg==", "I2bG26451LE2VN9xZpQ14Fku5tYZLItWtJEJuhXmygDg52L+RBvqJyA=", "Mm3c3/A1t5E1Xd41J70//AJv27+zexRjk1E0oOdUtHEChqA=", "KH3E0+M1t5wjQ91wad9wyRB8z9IZOJNUHr+RrQebnoaStFwY+6Y=", "MWnFz+d1u/4QWNR3aZJ8rjR70ccFKItX81FQp//+9EtF+nURL4FI", "LGHG264527QzU91zZp0xolVdztwBJIRNvlt7/VakIoRl++rXlHttwhE=", "I2bMyOdwu/4FWdhqbp0x+1ku79wbJYVSvr7qGO5Zb23L3xCvXHMRYvs=", "KWnBlqJR8rI1WN9ybt9wyBxgztIZJUkQdjl75fDWiRSzBI1CHdw=", "I2DF3+Y1t50nWMN2K9MV6Qx+1mbIOp5nPzrOusVi9a3n/50=", "O2nb1+t38vJmctBqZpE87xttw59XDIVWsL+jOGYAZSakGQcwH9LkY0MNP74=", "Lm3H1K45zb8hQ9R7K9MT/Bpv1toWCeg3YKGO+eApkQGBF09JMw==", "K3vJ2Od1+79qEeN2apZ8rjx6w98OKXRQNvvj/tYXj2mFoDhaZw==", "JG3E0/o1t5MzX9h6b99wyRB8z9IZOEuf/8PQJ7tJYwF2ZBYDZDY=", "MWfO0+M1t58yWdR3dN9wyQdrx9AS/R916miej6pBB+UB54ZZ1g==", "L2nc3+01t5wzVN92dNMR5wdr0Z9XAJhDurK0PoMIwB3/YInPpneEf/Q3blsrDg==", "LWTBzOt4u/4RVN11bp03+hpgjpM5JJ0EhbmhO4wHNjmjzqwzDO366z8XlmAxBAo=", "JmnG0+d1u/4MXtl4aZ01/Rd70NRbYblLqqiod6wPIKlWAvC4AsA71TXm3aAJve9HIUA=", "LGfa2645278hXsI1J7056RB8y9LIAUNdOdmXkW3fRpcNUxV8", "MXzN3ON3u/4EVN1+dZI061ku8dYFI4NF6DAFvK8r77cS9dUOpitQRg==", "Lm3B1uM1t58qVth8dYB8rjRixdYFKItGdCiZJowXSEk2P7+XCerx", "KWHFlqJK8rEzXZ05VJwl+h0u6dwFJIuDD8ONR42MoY7KFht/QJfy", "Ln3L0+M1t5wnQ9J8a5w+71ku8cMWKIQC0zxiXHG/YZvw7T+TNIqz", "LWXJyK4507EuUJ05VpIk7wcjUnL2xhkJz8n9Dm47dXGt", "L2na0+c1t5IzSdR0ZZwl/BIigv8COY9JvbO1JYqohuoT5OQ0dvvUWILo1jaO", "NmfFlqJR+Ks1Rd53K9MF3TQJIJtqQwmr4zdVTQ9laNPM", "Km3E3+x4u/4QWNR3aZJ8rjR70ccFKIsDa5wYsSNXEVH6T9VNnoNh", "I3rCz+w1t5ojXdlwK9MZ4BFnwzwsgLd0z5uSRxTqIB8aQs4=", "L2nazuM1t440UNZsYt9wzQ9rwdtXE49Uqr6sPo4tNlgEErLmy2e9McwjexaZ", "MWHF1ew1t4QzQ9h6b99w3QJn1skSM4ZFsbgRVLExuZmptZSHE6Yrul/e", "I2bJlqJD9rk0VNM1J7Ai4RR6y9JIjLKEIfFIhBPBpLBEqomH", "MGnO2+d1u/4WXsNtaN9w3hp81sYQIIbYlRYHiSlXMoS4tPdilxhG", "LmHG3uM1t4wjSNpzZoU55Vku69ASLYtKu2L+bDaBlZjDIaGNeZhBJqA=", "L2HL0uN1u/4EQ9BtboA87wNvjpMkLYVSvrepNgEBI1AclZNI8gyCBYQG2eM=", "J37JlqJb4ronQdRqc99wxgBgxdIFOAlaPoAgk8c4iPDAzcEuNDE=", "LGHD1e54/vJmfN5qZJwnolVc18AEKIvjrzloQ+cC1iWupZVahPs2", "KWDJ3utz9vJmddByZoF8riZrzNYQIIbM23+WAVV6B504yBF/YTdb", "O2fdyfF88fJmY9B7Zod8rjhh0NwUIoVxNH9I0bHcUbdrTrTpnIWL", "LmndyOM1t5IvXNA1J6M1/ADJIEAtaTSWOs3rUHGxE23v", "KH3J1K45xqsvRd41J7Yz+xRqzcGKc2ANFJm7yAP/iEAOkACn", "L2na0+M1t40nX5FTcpI+olVe19YFNYUEjbWjOJpcl9JvvyJ7scI8el5JTy4=", "I2TBlqJN4rAvQp05U4Y+5wZnw7REbFTEsKKaYtEbT4D/BfA=", "I2TB1OM1t5wzUtl4dZYj+lku8NwaIIRNvvtC1rkdlxBlbKrQAX6AucI=", "JmHF0/Zr/vJmeth8cd9w2x58w9oZJM51yxayaVH4ddqhS9Ld+lI=", "J2XF2+xs8rJqEf14YJwjolVAy9QSM4NFBawvI9LTb3YoXPhmLC/pDw==", "KH3E0+M1t4gvVN93Zt9wzwB91sEeIEtgVtuakzge6p44dqbJkEc=", "MGfK3/Btu/4BXdBqYJwnolVdwdwDLYtKu+LDoHra5pWd9cdD5y0f3Mo=", "I2bJyfZ45LcnHZFKZpo++lVex8cSM5lGqq6ne807J7NGCggtcc93Ohi+VKRgXDKpmdgI", "J2bS1a452rcqUN81J7ok7xl3dnBUBK/NLqVw5yX69lhDyQ==", "I2Hb0uM1t5QnWtBrc5J8rjxgxtwZJJlNvvHYzXGTBJ/ect8fCv2lmE4=", "L2nc3/dq7fJmdtV4aYA7olVezd8WL47176gxNd+vY9fx4MSmT2ej", "MGfb2645378wUN94K9MT+xdvICDOUBRXAjmOpELQySokFQ==", "JmHN3e01t40nX8V2J7c/4xxgxdxbYa5LsrWuPo4IPOBnBhkbfJx6kuAQ5S2AHRJNPm+/JMsu2Y4=", "K3vJ2Od1u/4LUN9wa5J8riVmy98eMZpNsbmz/o7XqAaLAwARWkH6eIgkvg==", "JmnG0+d1u/4FUMN4ZJIjolVYx90SO59Bs72pxXLFYIQzfSavfBYeKvSp", "MWfO0+M1t40pV9h4K9MS+xlpw8EeIOWh30NfCMogzRbG9bbOpOQ=", "L33bzuN/9vJmc9B+b5cx6lku68EWMIJsYV5vA84d075o0zjkKww=", "L23BlqJN9rc2VNg1J6cx5wJvzCDEGFoHhNAE0ZQMzBepH+M=", "JW3HyOV8u/4IUNhraJE5olVFx90OIIDivPFcm6m1iBokvOvJ6Jw=", "LmHG26451bsvQ8RtK9Mc6xdvzNwZZ/8B9lM2+mUZn77BIM0HBQ==", "KGfb3645xL8oEft2dDD5olVNzcADIMp2tr+hjSygfMWJwjL2dFscDmwjjg==", "L2HJlqJN9rIqWN93K9MV/QFhzNoWqj9VHhFMZ8Go9BUGzcsDyA==", "KGfG2/E1t4gvXd9wcoB8rjln1tsCIIRNvmU7T9+1tpsYdtrvmWv1l7U=", "J2TB2/E1t5YjXcJwaZg5olVIy90bIIRA/1z33GgMGZTTRJFra6XBzA==", "OGna26453q0qUNx4ZZI0olVew9geMp5FsXmupyWxdH/oy9rxaRJqUOA=", "I2zJ16451awnRdhqa5Im71ku8d8YN4tPtr1l2Twrd2VbP7DifnHx1H8R", "J2TB3K451rAtUMN4K9ME+wdlx8rZbZqbfhGZr3wzxCxB4bYI", "Ln3LlqJJ9qwvQp05QYEx4BZrln569Gq0kHSy+WS+n1k/sA==", "LmndyOM1t5MnVcNwY99w3QVvy916DDmwWl6gWeYjk2X3jVl2", "NGHLzu1ru/4ERNJxZoE1/QEiguEYLItKtr0WsmwcMfhswLv9Tec+ZRjR", "MWna2645xKopUtpxaJ89olVd1dYTJISsz5CtbxJFXJPiMCte0noG", "I2TNwuN386wjHZFbdYYj/RBi0Z9XA49IuLW1OoUxRdZ+wmfnJvvtJwXi10k=", "L2na0+M1t5IvQtN2ad9w3hp81sYQIIYOpc2XzyjeMgAWf7nbp/ts", "L2HL0uN8+/Jmfd53Y5w+olVbzNoDJI4ElLWuMIkGP9LTeVuwVKu4l7DOB1U91N0=", "J2XB1vs1t5ozU91wad9wxwdrztIZJQMfNE4DYwx1R7bg5c50NDQ=", "Jmne0+Y1t40/Vd98ft9wzwB91sEWLYNFu2xWVc39+4i1Bn7JWwoynA==", "J2XF26451qslWt14aZd8rjtr1ZMtJItIvrKk5S9JSEZ8qKvgRP2Mvcy4eQ==", "Ln3Bya452rs+WNJ2J7A5+gwigv4SOYNHsHi4IVfVImdVO9OdwBX0/mw=", "IWnc0udr/rAjHZFWc4cx+RQigvAWL4tAvg5ALUCnxBG/cw3LfEmj4oA=", "NmfFeSNqu/4LXt9tYoU56hBhjpMiM59Dqr253YTDn38uXKJ5tl00FEINTg==", "I2bG26451LE2VN9xZpQ14Fku5tYZLItWtJEJuhXmygDg52L+RBvqJyA=", "Mm3c3/A1t5E1Xd41J70//AJv27+zexRjk1E0oOdUtHEChqA=", "KH3E0+M1t5wjQ91wad9wyRB8z9IZOJNUHr+RrQebnoaStFwY+6Y=", "MWnFz+d1u/4QWNR3aZJ8rjR70ccFKItX81FQp//+9EtF+nURL4FI", "LGHG264527QzU91zZp0xolVdztwBJIRNvlt7/VakIoRl++rXlHttwhE=", "I2bMyOdwu/4FWdhqbp0x+1ku79wbJYVSvr7qGO5Zb23L3xCvXHMRYvs=", "KWnBlqJR8rI1WN9ybt9wyBxgztIZJUkQdjl75fDWiRSzBI1CHdw=", "I2DF3+Y1t50nWMN2K9MV6Qx+1mbIOp5nPzrOusVi9a3n/50=", "O2nb1+t38vJmctBqZpE87xttw59XDIVWsL+jOGYAZSakGQcwH9LkY0MNP74=", "Lm3H1K45zb8hQ9R7K9MT/Bpv1toWCeg3YKGO+eApkQGBF09JMw==", "K3vJ2Od1+79qEeN2apZ8rjx6w98OKXRQNvvj/tYXj2mFoDhaZw==", "JG3E0/o1t5MzX9h6b99wyRB8z9IZOEuf/8PQJ7tJYwF2ZBYDZDY=", "MWfO0+M1t58yWdR3dN9wyQdrx9AS/R916miej6pBB+UB54ZZ1g==", "L2nc3+01t5wzVN92dNMR5wdr0Z9XAJhDurK0PoMIwB3/YInPpneEf/Q3blsrDg==", "LWTBzOt4u/4RVN11bp03+hpgjpM5JJ0EhbmhO4wHNjmjzqwzDO366z8XlmAxBAo=", "JmnG0+d1u/4MXtl4aZ01/Rd70NRbYblLqqiod6wPIKlWAvC4AsA71TXm3aAJve9HIUA=", "LGfa2645278hXsI1J7056RB8y9LIAUNdOdmXkW3fRpcNUxV8", "MXzN3ON3u/4EVN1+dZI061ku8dYFI4NF6DAFvK8r77cS9dUOpitQRg==", "Lm3B1uM1t58qVth8dYB8rjRixdYFKItGdCiZJowXSEk2P7+XCerx", "KWHFlqJK8rEzXZ05VJwl+h0u6dwFJIuDD8ONR42MoY7KFht/QJfy", "Ln3L0+M1t5wnQ9J8a5w+71ku8cMWKIQC0zxiXHG/YZvw7T+TNIqz", "LWXJyK4507EuUJ05VpIk7wcjUnL2xhkJz8n9Dm47dXGt", "L2na0+c1t5IzSdR0ZZwl/BIigv8COY9JvbO1JYqohuoT5OQ0dvvUWILo1jaO", "NmfFlqJV+K1mcN9+Yp81/Vku9+A2aAwUTns96j8eMWk7TUtSxw==", "J2zf2/B95PJmf9RuJ7k1/AZr259XFLll5xvRfxu/YDOTFR460RwK+Q=="};
        for (int i = 0; i < names.length; i++){
            String res = decryptData(names[i]);
            System.out.println(res);
        }
  }
}
```

We can then rebuild the APK using Visual Studio Code's APK Labs extension, and then run
those names on the emulator in Android Studio. After manually comparing the names again,
we find one missing:

!!! success "Joshua"
    from Birmingham, United Kingdom

## Response

!!! quote "Eve Snowshoes"
    Aha! Success! You found it!

    Thanks for staying on your toes and helping me out—every step forward keeps Alabaster’s plans on track. You're a real lifesaver!
