# Java Reflections
  ## Section 1
  - what is java reflection ?
    * Language and jvm features that gives us Runtime access to information about our application's classes and objects.
    * Those features are available to us through the Java reflection API

  - What can we do with java Reflection ?
    * Using Reflection Api we can write flexible code that
      -> Links different software components together at runtime
      -> Creates new program flow without any source code modifications
    * Reflection allows writing general purpose alogorithms that dynamically adapt and change their behaviour based on the type of objects or classes they are
      working on.
    * with the ability to analyse applications objects and classes at runtime
    * we can create very powerful
      -> libraries
      -> Frameworks
      -> Software Designs that would otherwise be impossible

  - Reflections Apis used in
    * Junit(custom annotation)
    * Dependency Injection(DI)
      -> Spring/Spring boot
      -> Google Giuce(DI)
    * JSON Serialization/Deserialization
      -> Jackson
      -> Gson
    * Other popular use cases include
      -> Logging Frameworks
      -> ORM
      -> Web Frameworks
      -> Developer Tools ... etc

  - Reflection Api Gateway & WildCards(section 1 - 2)
    * Reflection Entry Point - Class<?>
      -> Class<?> is an entry point for us to reflect on our applications code
      -> An Object of Class<?> contains all the information on
        1) A given objects runtime type
        2) A class in our application
      -> That information includes
        1) What methods and fields it contains
        2) What classes it extends
        3) what interfaces it implements
    * How to get the Object os a class
    
  ```
      -> Object.getClass()
        String stringObject = "some-string";
        Car car = new Car();
        Map<String, Integer> map = new HashMap<String, Integer>();

        Class<String> stringClass = stringObject.getClass();
        Class<Car> carClass = car.getClass();
        Class<?> mapClsss = map.getClass(); // Return HashMap class and not HashMap coz it is an interface

        --------------

        No Object.getClass(); for primitive types
        boolean condition = true;
        int value = 55;
        double price = 1.5;

        Class booleanClass = condition.getClass(); // compilation error
        Class intClass = int.getClass(); // compilation error
        Class doublePrice = price.getClass(); compilation error

      -> ".class" suffix to a type name
        --> used when we don't have a instance of a class, but we need to get information of the class itself(static)
          Class<String> stringClass = String.class;
          Class<Car> carClass = Car.class;
          Class<?> mapClass = HashMap.class;

          ---------

          ".class" can be used to check class for primitive types

          Class booleanType = boolean.class;
          Class intType = int.class;
          Class doubleType = double.class;

      -> Class.forName(...)
          --> we can dynamically look up any class in the class path using it's fullly qualified name. That includes built in classes also like "String"

            Class<?> stringType = Class.forName("java.lang.String");
            Class<?> carType = Class.forName("vechile.car");
            // even inner clases can be accessed
            Class<?> engintType = Class.forName("vechile.Car$Engine");

            Package vechile;

            Class Car {
              ....
              static class Engine {

              }
              ...
            }

            ----------------

            cannot use "Class.forName(...)" to get the class of primitive types

              boolean condition = true;
              int value = 55;
              double price = 1.5;
              // note: see the error diff for Class.forName and Object.getClass()
              Class booleanClass = Class.forName("boolean"); // Runtime Error
              Class intClass = Class.forName("int"); // Runtime Error
              Class doublePrice = Class.forName("double"); // Runtime Error
```
          --> Much more likely to mistype the class name and get the ClassNotFoundException
          --> The Class we pass into the Class.forName(...) may not even exist
          --> Class.forName(...) is the least safest way to get a Class<?> object
          --> There are use cases where Class.forNmae(...) is our only option
          --> use case
            a) when the class name is in the config file and we need to instantate the class at runtime
            b) the class we need to instantiate is not a part of the code base, but is added at runtime, like external libraries

        -> <?> Generics/wildcards
          --> Using Class<?> we can describe a Class Object of any parameter type
          --> Class<?> is a super type to Class<T> of any type T
            Class<?> carClass = Class.forName("vechile.Car");
            Map<String, Integer> genericMap = new HashMap<>();
            Class<?> hashMapClass = genericMap.getClass();
            // Restrict to only types that extend collection
            public List<String> findAllMethods(Class<? extends collection> clazz) {
              ...
            }

    * Implementaion example for above(Section 1 -3)
      - check if a class is primitive, enum, anonmous, interface or array
              inputClass.isPrimitive()
              inputClass.isInterface()
              inputClass.isEnum()
              inputClass.getSimpleName()
            isJdkClass(inputClass) // custom implementation
      - get the class name
      - get the interfaces implemented
      - getPackage
      - isJDKClass
            public static boolean isJdkClass(Class<?> inputClass) {
              return JDK_PACKAGE_PREFIXES.stream()
                        .anyMatch(packagePrefix -> inputClass.getPackage() == null
                                    || inputClass.getPackage().getName().startsWith(packagePrefix));
            }
      - get all classes that implement a interface
          if (inputClass.isInterface()) {
            inheritedClasses = Arrays.stream(inputClass.getInterfaces())
                    .map(Class::getSimpleName)
                    .toArray(String[]::new);
          }
      - get all the super classes
         Class<?> inheritedClass = inputClass.getSuperclass();
      - program

            package exercises;
            import java.util.Collection;
            import java.util.HashMap;
            import java.util.Map;

            /**
            * Class Demonstration
            * https://www.udemy.com/course/java-reflection-master-class
            */
            public class Main {

                public static void main(String[] args) throws ClassNotFoundException {
                    Class<String> stringClass = String.class;

                    Map<String, Integer> mapObject = new HashMap<>();

                    Class<?> hashMapClass = mapObject.getClass();

                    Class<?> squareClass = Class.forName("exercises.Main$Square");

                    printClassInfo(stringClass, hashMapClass, squareClass);

                    var circleObject = new Drawable() {
                        @Override
                        public int getNumberOfCorners() {
                            return 0;
                        }
                    };


                    printClassInfo(Collection.class, boolean.class, int[][].class, Color.class, circleObject.getClass());
                }

                private static void printClassInfo(Class<?>... classes) {

                    for (Class<?> clazz : classes) {

                        System.out.println(String.format("class name : %s, class package name : %s",
                                clazz.getSimpleName(),
                                clazz.getPackageName()));

                        Class<?>[] implementedInterfaces = clazz.getInterfaces();

                        for (Class<?> implementedInterface : implementedInterfaces) {
                            System.out.println(String.format("class %s implements : %s",
                                    clazz.getSimpleName(),
                                    implementedInterface.getSimpleName()));
                        }

                        System.out.println("Is array : " + clazz.isArray());
                        System.out.println("Is primitive : " + clazz.isPrimitive());
                        System.out.println("Is enum : " + clazz.isEnum());
                        System.out.println("Is interface : " + clazz.isInterface());
                        System.out.println("Is anonymous :" + clazz.isAnonymousClass());

                        System.out.println();
                        System.out.println();
                    }
                }

                private enum Color {
                    BLUE,
                    RED,
                    GREEN
                }

                private static interface Drawable {
                    int getNumberOfCorners();
                }

                static class Square implements Drawable {

                    @Override
                    public int getNumberOfCorners() {
                        return 4;
                    }
                }

            }
      - program to find all the interfaces implemented by a class using recurssion
        public class Exercise {
            /**
            * Returns all the interfaces that the current input class implements
            * Note: If the input is an interface, the method returns all the interfaces the
            * input interfaces extends
            */
            public static Set<Class<?>> findAllImplementedInterfaces(Class<?> input) {
                Set<Class<?>> allImplementedInterfaces = new HashSet<>();

                Class<?>[] inputInterfaces = input.getInterfaces();
                for (Class<?> currentInterface : inputInterfaces) {
                    allImplementedInterfaces.add(currentInterface);
                    allImplementedInterfaces.addAll(findAllImplementedInterfaces(currentInterface));
                }

                return allImplementedInterfaces;
            }
        }

  ## Section 2
    * Object Creation and Constructors
      - A constructor of a java class is represented by an instance of COnstructor<?> class
      - The Constructor object contains all the information about the class's constructor including:
             -> Number of parameters
             -> Types of the parameters
      - A Class may have multiple constructors         
      - Methods to get Constructors <?> objects
        -> Returns all declared constructors within the class
        -> Includes all the public and non-public constructors
      - Class.getConstructors()
        -> return only public constructors of the class
      - If we know the particular constructor based on the parameter types we call:  
        -> Class.getConstructor(Class<?> ...parameterTypes) or Class.getDeclaredConstructors(Class<?> ...parameterTypes)
           --> returns a particular constructor based on the parameter types
           --> Throws NoSuchMethodException if no matching constructoro is found

      ```
        Getting default constructor
        public class Person {
          // no declared constructors
          public Person() {}  // -> Auto Generated default constructor
        }
        Constructors<?>[] constructors = Person.class.getConstructors(); // Single element array containing the default constructor
        Constructors<?>[] constructors = Person.class.getDeclaredConstructors(); // Single element array containing the default constructor

        // To get parameters of the constructors
        
        for(int i = 0; i < constructors.length; i++) {
        Class<?>[] parametersTypes = constructors[i].getParameterTypes();
        List<String> parameterTypeNames = Array.stream(parametersTypes)
                                          .map(type -> type.getSimpleName())
                                          .collect(Collectors.toList());                                                
        }
      ```     

      - To create an object of a class using above method
        -> Implement a single "factory" method that can create an object of any class
        -> Depending on the arguments passed to our method, it will find the right constructor
        -> Create the given class object by calling the right constructor
        -> withour reflection it is impossible

        ```
         without using reflection

         public Object createObject(Typw typw, Object arg) {
           switch(type) {
           case Type.Employee: return new Employee(arg);
           }
         }
        ```
      
     
