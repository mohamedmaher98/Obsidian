\# Calculating with Math APIs in Java

## ✔️ Overview
Java provides the `Math` class for performing common mathematical calculations:
- Max/Min
- Absolute value
- Power and roots
- Rounding
- Random numbers
- Trigonometry (sin, cos, tan)
- Logarithms
- Constants like PI and E

---

## ✔️ Common Methods

### **1. Max & Min**
```java
Math.max(a, b);
Math.min(a, b);
```

### **2. Absolute Value**
```java
Math.abs(-5); // 5
```

### **3. Power**
```java
Math.pow(2, 3); // 8
```

### **4. Square Root**
```java
Math.sqrt(25); // 5
```

### **5. Random Numbers**
```java
double r = Math.random(); // 0.0 to <1.0
int num = (int)(Math.random() * 100); // 0–99
```

---

## ✔️ Rounding Methods

### **round() – nearest whole**
```java
Math.round(4.4); // 4
Math.round(4.5); // 5
```

### **ceil() – round up**
```java
Math.ceil(4.1); // 5
```

### **floor() – round down**
```java
Math.floor(4.9); // 4
```

---

## ✔️ Trigonometry (Radians)
```java
Math.sin(Math.toRadians(30)); // 0.5
Math.cos(Math.toRadians(60)); // 0.5
Math.tan(Math.toRadians(45)); // 1
```

---

## ✔️ Logarithms
```java
Math.log(10);
Math.log10(100); // 2
```

---

## ✔️ Constants
```java
Math.PI; // 3.141592...
Math.E;  // 2.71828...
```

---

## ✔️ Notes
- Trig functions use **radians**, not degrees.
- `Math.random()` returns a double between **0.0 and <1.0**.
