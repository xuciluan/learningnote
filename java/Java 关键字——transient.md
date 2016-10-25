# Java 关键字——transient

## 1. transient的作用

transient提供了一种阻止变量被序列化的方法。我们知道，当一个类需要序列化的时候，只需要实现`Serilizable`接口即可。但是，在实际使用过程中，我们可能要求只序列化类的一些属性，而类的另外一些属性不被序列化，此时，为了实现这个功能，我们只需要在不想被序列化的属性前加上`transient`进行修饰即可。

举例：（例子摘抄自[transient关键字](http://www.cnblogs.com/lanxuezaipiao/p/3369962.html))

```java
import java.io.FileInputStream;
import java.io.FileOutputStream;
import java.io.IOException;
import java.io.ObjectInputStream;
import java.io.ObjectOutputStream;
import java.io.Serializable;

public class TransientTest {

	public static void main(String[] args) {
		User user = new User();
		user.setUserName("Alexia");
		user.setPasswd("1235");
		
		   System.out.println("read before Serializable: ");
	        System.out.println("username: " + user.getUserName());
	        System.err.println("password: " + user.getPasswd());
	        
	        
	       
	        try {
	        	ObjectOutputStream os = new ObjectOutputStream(
	        		new FileOutputStream("D:/user.txt"));
				os.writeObject(user);
				os.flush();
				os.close();
			} catch (IOException e) {
				// TODO Auto-generated catch block
				e.printStackTrace();
			}
	        
	        
	        
	        ObjectInputStream is;
			try {
				is = new ObjectInputStream
						(new FileInputStream("D:/user.txt"));
			user = (User) is.readObject();
			System.out.println("\nread after Serializable: ");
            System.out.println("username: " + user.getUserName());
            System.err.println("password: " + user.getPasswd());
			} catch (IOException e) {
				// TODO Auto-generated catch block
				e.printStackTrace();
			} catch (ClassNotFoundException e) {
				// TODO Auto-generated catch block
				e.printStackTrace();
			}
	        
				
	}
}

class User implements Serializable{
	
    /**
	 * 
	 */
	private static final long serialVersionUID = 1L;
	private String userName;
    private transient String passwd;
	public String getUserName() {
		return userName;
	}
	public void setUserName(String userName) {
		this.userName = userName;
	}
	public String getPasswd() {
		return passwd;
	}
	public void setPasswd(String passwd) {
		this.passwd = passwd;
	}
    
    
}
```

## 2. transient使用小结

- 一个变量一旦被transient修饰，它将不再是对象持久化的一部分，该变量内容在序列化后无法访问

- transient关键字只能修饰变量，不能修饰方法和类。**变量如果是用户自定义的，则需要实现Serializable接口。**

- 一个静态变量不管是否被transient修饰，均不能被序列化

- 若类实现的是`Externalizable`接口，那么不管它的属性是否被transient修饰，它都会被序列化，因为若实现的是Externalizable接口，则没有任何东西可以自动序列化，需要在writeExternal方法中进行手工指定所要序列化的变量，这与是否被transient修饰无关。

  举例如下：

  ```java
  import java.io.Externalizable;
  import java.io.File;
  import java.io.FileInputStream;
  import java.io.FileOutputStream;
  import java.io.IOException;
  import java.io.ObjectInput;
  import java.io.ObjectInputStream;
  import java.io.ObjectOutput;
  import java.io.ObjectOutputStream;

  /**
   * @descripiton Externalizable接口的使用
   * 
   * @author Alexia
   * @date 2013-10-15
   *
   */
  public class ExternalizableTest implements Externalizable {

      private transient String content = "是的，我将会被序列化，不管我是否被transient关键字修饰";

      @Override
      public void writeExternal(ObjectOutput out) throws IOException {
          out.writeObject(content);
      }

      @Override
      public void readExternal(ObjectInput in) throws IOException,
              ClassNotFoundException {
          content = (String) in.readObject();
      }

      public static void main(String[] args) throws Exception {
          
          ExternalizableTest et = new ExternalizableTest();
          ObjectOutput out = new ObjectOutputStream(new FileOutputStream(
                  new File("test")));
          out.writeObject(et);

          ObjectInput in = new ObjectInputStream(new FileInputStream(new File(
                  "test")));
          et = (ExternalizableTest) in.readObject();
          System.out.println(et.content);

          out.close();
          in.close();
      }
  }
  ```

  ​

  ​