# GoF Design Patterns

Catatan ringkas tentang beberapa **Gang of Four (GoF) Design Patterns** yang dipelajari.

> Fokus utama: memahami **masalah yang diselesaikan pattern**, bukan menghafal bentuk class secara mentah.

---

# 1. Singleton

## Pengertian

**Singleton** memastikan sebuah class hanya memiliki **satu instance** dan menyediakan akses global ke instance tersebut.

```java
class Database {

    private static Database instance;

    private Database() {}

    public static Database getInstance() {
        if (instance == null) {
            instance = new Database();
        }

        return instance;
    }
}
```

Usage:

```java
Database db1 = Database.getInstance();
Database db2 = Database.getInstance();

System.out.println(db1 == db2); // true
```

### Inti

```text
Satu class
   ↓
Satu instance
   ↓
Digunakan bersama
```

### Catatan FP

Dalam Functional Programming, Singleton class biasanya tidak diperlukan. Dependency dapat dibuat sekali di composition root lalu diberikan melalui dependency injection.

---

# 2. Builder

## Pengertian

**Builder** digunakan untuk membangun object yang kompleks secara bertahap.

```java
class User {

    String name;
    String email;
    int age;

    UserBuilder setName(String name) {
        this.name = name;
        return this;
    }

    UserBuilder setEmail(String email) {
        this.email = email;
        return this;
    }

    UserBuilder setAge(int age) {
        this.age = age;
        return this;
    }

    User build() {
        return new User(name, email, age);
    }
}
```

Usage:

```java
User user = new UserBuilder()
    .setName("Ubay")
    .setEmail("ubay@example.com")
    .setAge(20)
    .build();
```

### Kenapa `return this`?

Agar method bisa di-chain:

```java
builder
    .setName(...)
    .setEmail(...)
    .setAge(...);
```

`this` adalah instance builder yang sedang digunakan.

### Inti

```text
Object kompleks
      ↓
Dibangun bertahap
      ↓
build()
      ↓
Object jadi
```

### Kata kunci

**Step-by-step object construction**

---

# 3. Simple Factory

## Pengertian

**Simple Factory** adalah satu object/function yang bertugas menentukan concrete object yang harus dibuat.

```java
class AnimalFactory {

    static Animal create(String type) {

        if (type.equals("dog")) {
            return new Dog();
        }

        if (type.equals("cat")) {
            return new Cat();
        }

        throw new IllegalArgumentException();
    }
}
```

Usage:

```java
Animal animal = AnimalFactory.create("dog");
```

Struktur:

```text
Factory
   ↓
create()
   ↓
Concrete Product
```

### Catatan

Simple Factory **bukan GoF Factory Method** secara formal.

---

# 4. Factory Method

## Pengertian

**Factory Method** mendefinisikan method untuk membuat object, tetapi concrete subclass menentukan object apa yang dibuat.

```java
abstract class AnimalCreator {

    abstract Animal createAnimal();

    void makeSound() {
        Animal animal = createAnimal();
        animal.makeSound();
    }
}
```

Concrete creator:

```java
class DogCreator extends AnimalCreator {

    Animal createAnimal() {
        return new Dog();
    }
}
```

```java
class CatCreator extends AnimalCreator {

    Animal createAnimal() {
        return new Cat();
    }
}
```

Usage:

```java
AnimalCreator creator = new DogCreator();

creator.makeSound();
```

Struktur:

```text
AnimalCreator
      ↓
createAnimal()
      ↑
      │
 ┌────┴─────┐
DogCreator  CatCreator
    ↓           ↓
   Dog         Cat
```

### Inti

> **Subclass menentukan concrete object yang dibuat.**

### Kata kunci

**Object creation delegated to subclass**

---

# 5. Abstract Factory

## Pengertian

**Abstract Factory** menyediakan interface untuk membuat **keluarga product yang saling berkaitan**.

Contoh:

```java
interface UIFactory {

    Button createButton();

    Checkbox createCheckbox();
}
```

Windows:

```java
class WindowsFactory implements UIFactory {

    public Button createButton() {
        return new WindowsButton();
    }

    public Checkbox createCheckbox() {
        return new WindowsCheckbox();
    }
}
```

Web:

```java
class WebFactory implements UIFactory {

    public Button createButton() {
        return new WebButton();
    }

    public Checkbox createCheckbox() {
        return new WebCheckbox();
    }
}
```

Struktur:

```text
                UIFactory
              /           \
             /             \
WindowsFactory             WebFactory
    ↓   ↓                     ↓   ↓
WindowsButton           WebButton
WindowsCheckbox         WebCheckbox
```

### Factory Method vs Abstract Factory

```text
Factory Method
→ membuat satu jenis product

Abstract Factory
→ membuat satu keluarga product terkait
```

Abstract Factory bisa menggunakan beberapa Factory Method di dalamnya.

### Kata kunci

**Family of related products**

---

# 6. Prototype

## Pengertian

**Prototype** membuat object baru dengan menyalin object yang sudah ada sebagai template.

```java
class Character {

    String name;
    int health;

    Character copy() {
        Character copy = new Character();

        copy.name = this.name;
        copy.health = this.health;

        return copy;
    }
}
```

Usage:

```java
Character original = new Character();

original.name = "Warrior";
original.health = 100;

Character copy = original.copy();
```

Struktur:

```text
Existing Object
      ↓
    copy()
      ↓
New Object
```

---

## Shallow Copy

Object luar dicopy, tetapi object yang direferensikan masih sama.

```text
original ──┐
           ├──→ Weapon
copy ──────┘
```

```java
original.weapon == copy.weapon; // true
```

---

## Deep Copy

Object luar dan object yang direferensikan ikut dicopy.

```text
original ──→ Weapon A

copy ──────→ Weapon B
```

```java
original.weapon == copy.weapon; // false
```

### Inti

> **Existing object dijadikan template untuk membuat object baru.**

### Kata kunci

**Cloning / copying existing object**

---

# 7. Adapter

## Pengertian

**Adapter** membuat interface yang tidak cocok menjadi kompatibel dengan interface yang dibutuhkan client.

Misalnya client membutuhkan:

```java
interface Payment {
    void pay(int amount);
}
```

Tetapi library yang tersedia:

```java
class Stripe {

    void makePayment(double amount) {
        System.out.println("Paid");
    }
}
```

Buat adapter:

```java
class StripeAdapter implements Payment {

    private Stripe stripe;

    StripeAdapter(Stripe stripe) {
        this.stripe = stripe;
    }

    public void pay(int amount) {
        stripe.makePayment((double) amount);
    }
}
```

Usage:

```java
Payment payment = new StripeAdapter(new Stripe());

payment.pay(100);
```

Struktur:

```text
Client
  ↓
Payment
  ↓
Adapter
  ↓
Stripe
```

### Inti

> **Interface tidak cocok → diterjemahkan oleh Adapter.**

### Kata kunci

**Compatibility / interface translation**

---

# 8. Facade

## Pengertian

**Facade** menyediakan interface sederhana untuk mengakses subsystem yang kompleks.

Tanpa Facade:

```text
Client
 ├── PaymentService
 ├── InventoryService
 ├── ShippingService
 └── NotificationService
```

Dengan Facade:

```text
Client
  ↓
OrderFacade
  ↓
 ├── PaymentService
 ├── InventoryService
 ├── ShippingService
 └── NotificationService
```

Contoh:

```java
class OrderFacade {

    private PaymentService payment;
    private InventoryService inventory;
    private ShippingService shipping;

    void placeOrder() {
        payment.charge();
        inventory.reduceStock();
        shipping.createShipment();
    }
}
```

Client:

```java
orderFacade.placeOrder();
```

Client tidak perlu tahu seluruh detail subsystem.

### Inti

> **Sistem kompleks → diberikan interface sederhana.**

### Kata kunci

**Simplify complex subsystem**

### Contoh di aplikasi backend

Service yang mengorkestrasi beberapa repository/service bisa berperan sebagai Facade.

```text
Controller
    ↓
OrderService
    ↓
 ├── OrderRepository
 ├── PaymentService
 ├── InventoryRepository
 └── NotificationService
```

Namun **Service tidak otomatis berarti Facade**. Yang menentukan adalah tujuan desainnya.

---

# 9. Template Method

## Pengertian

**Template Method** mendefinisikan kerangka algoritma di parent class, sementara subclass mengimplementasikan langkah tertentu.

```java
abstract class Beverage {

    public final void prepare() {
        boilWater();
        prepareIngredient();
        pour();
        serve();
    }

    void boilWater() {
        System.out.println("Boiling water");
    }

    abstract void prepareIngredient();

    void pour() {
        System.out.println("Pouring water");
    }

    void serve() {
        System.out.println("Serving");
    }
}
```

Coffee:

```java
class Coffee extends Beverage {

    void prepareIngredient() {
        System.out.println("Adding coffee");
    }
}
```

Tea:

```java
class Tea extends Beverage {

    void prepareIngredient() {
        System.out.println("Adding tea");
    }
}
```

Alurnya:

```text
prepare()
   ↓
boilWater()
   ↓
prepareIngredient()  ← subclass menentukan
   ↓
pour()
   ↓
serve()
```

### Inti

> **Parent menentukan alur, subclass menentukan detail langkah tertentu.**

### Kata kunci

**Fixed algorithm skeleton + customizable steps**

---

# 10. Bridge

## Pengertian

**Bridge** memisahkan abstraction dan implementation agar keduanya dapat berkembang secara independen.

Contoh:

```java
interface Device {

    void turnOn();

    void turnOff();
}
```

Concrete implementation:

```java
class TV implements Device {

    public void turnOn() {
        System.out.println("TV ON");
    }

    public void turnOff() {
        System.out.println("TV OFF");
    }
}
```

Abstraction:

```java
class Remote {

    protected Device device;

    Remote(Device device) {
        this.device = device;
    }

    void powerOn() {
        device.turnOn();
    }

    void powerOff() {
        device.turnOff();
    }
}
```

Usage:

```java
Device tv = new TV();

Remote remote = new Remote(tv);

remote.powerOn();
```

Struktur:

```text
Abstraction              Implementation

Remote ───────────────→ Device
  │                       │
  ├── BasicRemote         ├── TV
  └── AdvancedRemote      ├── Radio
                          └── AC
```

Tanpa Bridge, kombinasi subclass dapat meledak:

```text
BasicTVRemote
BasicRadioRemote
BasicACRemote

AdvancedTVRemote
AdvancedRadioRemote
AdvancedACRemote
```

Bridge memisahkan dua hierarchy tersebut.

### Inti

> **Pisahkan dua dimensi yang sama-sama dapat berkembang.**

### Kata kunci

**Decouple abstraction from implementation**

---

# 11. Composite

## Pengertian

**Composite** memungkinkan object individual dan kumpulan object diperlakukan melalui interface yang sama.

Contoh file system:

```text
Root
├── file.txt
├── photo.jpg
└── Documents
    ├── thesis.pdf
    └── notes.txt
```

Interface:

```java
interface FileSystem {
    void show();
}
```

Leaf:

```java
class File implements FileSystem {

    private String name;

    File(String name) {
        this.name = name;
    }

    public void show() {
        System.out.println(name);
    }
}
```

Composite:

```java
class Folder implements FileSystem {

    private String name;
    private List<FileSystem> children = new ArrayList<>();

    Folder(String name) {
        this.name = name;
    }

    void add(FileSystem item) {
        children.add(item);
    }

    public void show() {
        System.out.println(name);

        for (FileSystem child : children) {
            child.show();
        }
    }
}
```

Struktur:

```text
              FileSystem
             /          \
            /            \
         File           Folder
                         /  \
                        /    \
                     File   Folder
```

`File` = **Leaf**

`Folder` = **Composite**

Keduanya menggunakan interface yang sama.

### Inti

> **Individual object dan group of objects diperlakukan secara seragam.**

### Kata kunci

**Tree structure / part-whole hierarchy**

---

# Quick Comparison

| Pattern          | Masalah yang diselesaikan                             |
| ---------------- | ----------------------------------------------------- |
| Singleton        | Memastikan satu instance                              |
| Builder          | Membangun object kompleks secara bertahap             |
| Simple Factory   | Memusatkan pemilihan object yang dibuat               |
| Factory Method   | Subclass menentukan concrete product                  |
| Abstract Factory | Membuat keluarga product terkait                      |
| Prototype        | Membuat object dengan cloning                         |
| Adapter          | Membuat interface yang tidak cocok menjadi kompatibel |
| Facade           | Menyederhanakan subsystem kompleks                    |
| Template Method  | Menentukan skeleton algoritma                         |
| Bridge           | Memisahkan abstraction dan implementation             |
| Composite        | Memperlakukan leaf dan group secara seragam           |

---

# Pattern yang Sering Terlihat Mirip

## Strategy vs Bridge

### Strategy

```text
Context
   ↓
Strategy
   ├── Strategy A
   ├── Strategy B
   └── Strategy C
```

Fokus:

> **Menukar behavior/algorithm.**

### Bridge

```text
Abstraction
   ↓
Implementation
   ├── Implementation A
   ├── Implementation B
   └── Implementation C
```

Fokus:

> **Memisahkan dua hierarchy yang dapat berkembang independen.**

Keduanya bisa menggunakan composition + dependency injection, tetapi **intent-nya berbeda**.

---

# Adapter vs Facade

## Adapter

```text
Client
  ↓
Adapter
  ↓
Existing Class
```

> Interface tidak cocok → diterjemahkan.

## Facade

```text
Client
  ↓
Facade
  ↓
Subsystem A
Subsystem B
Subsystem C
```

> Interface terlalu kompleks → disederhanakan.

---

# Bridge + Adapter + Factory Method

Pattern bisa digunakan bersama.

Misalnya:

```text
Remote
   │
   │ Bridge
   ↓
Device
   ↑
SamsungTVAdapter
   │
   │ Adapter
   ↓
SamsungTV
```

Factory Method bahkan bisa digunakan untuk menentukan concrete `Device`:

```text
createDevice()
     ↓
SamsungTVAdapter
```

Jadi:

> **Pattern tidak harus berdiri sendiri. Satu sistem nyata dapat menggunakan beberapa pattern sekaligus.**

Yang harus diperhatikan adalah **problem yang diselesaikan oleh masing-masing pattern**, bukan sekadar bentuk class-nya.

---

# Functional Programming Perspective

Beberapa pattern GoF sangat berhubungan dengan OOP karena GoF dirancang untuk problem object-oriented.

Dalam FP, beberapa struktur bisa dibuat jauh lebih sederhana.

Contoh Factory:

```javascript
const animals = {
    dog: () => ({
        type: "dog",
        sound: "woof"
    }),

    cat: () => ({
        type: "cat",
        sound: "meow"
    })
};

const createAnimal = (type) => animals[type]();
```

Usage:

```javascript
const dog = createAnimal("dog");

console.log(dog.type);
console.log(dog.sound);
```

Tidak diperlukan:

```text
Animal interface
AnimalCreator
DogCreator
CatCreator
```

Karena function dapat menjadi value dan composition dapat menggantikan sebagian penggunaan inheritance/polymorphism.

---

# Cara Menghafal GoF

Jangan hafalkan:

```text
"Pattern X harus punya abstract class Y
dan interface Z."
```

Lebih baik hafalkan:

```text
PROBLEM
   ↓
INTENT
   ↓
STRUCTURE
```

Contoh:

```text
Adapter
Interface tidak cocok
        ↓
Terjemahkan interface
        ↓
Adapter
```

```text
Facade
Subsystem terlalu kompleks
        ↓
Sederhanakan akses
        ↓
Facade
```

```text
Bridge
Dua dimensi berkembang independen
        ↓
Pisahkan abstraction & implementation
        ↓
Bridge
```

```text
Composite
Object individual + group
        ↓
Perlakukan secara seragam
        ↓
Tree / Composite
```

```text
Template Method
Algoritma punya alur tetap
        ↓
Parent menentukan skeleton
        ↓
Subclass mengisi langkah tertentu
```

```text
Factory Method
Concrete object berbeda
        ↓
Subclass menentukan object yang dibuat
        ↓
Factory Method
```

```text
Abstract Factory
Butuh family of related products
        ↓
Factory membuat satu keluarga
        ↓
Abstract Factory
```

```text
Prototype
Object kompleks sudah tersedia
        ↓
Clone object existing
        ↓
Prototype
```

```text
Builder
Object kompleks dibuat bertahap
        ↓
Step-by-step construction
        ↓
Builder
```

```text
Singleton
Butuh satu instance
        ↓
Satu instance
        ↓
Singleton
```

---

# One-Line Cheat Sheet

```text
Singleton
→ satu instance

Builder
→ bangun object bertahap

Factory Method
→ subclass menentukan object yang dibuat

Abstract Factory
→ buat family of related products

Prototype
→ clone object yang sudah ada

Adapter
→ terjemahkan interface yang tidak cocok

Facade
→ sederhanakan subsystem yang kompleks

Template Method
→ parent menentukan alur, subclass mengisi langkah

Bridge
→ pisahkan abstraction dan implementation

Composite
→ leaf + group diperlakukan sama dalam tree
```

---

# Core Principle

Pada akhirnya, jangan melihat Design Pattern sebagai **template kode yang wajib disalin**.

Design Pattern lebih tepat dipahami sebagai:

> **solusi desain yang berulang untuk problem tertentu.**

Kode implementasinya dapat berbeda berdasarkan bahasa, framework, arsitektur, dan kebutuhan sistem.
