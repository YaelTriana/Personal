# Adquisicipon de datos seriales de temperatura de una microbit v2 con diferentes lenguajes
## Autor: Yael Kristoph Triana Sánchez
--- 
## Python
    import serial
    import time

    puerto = 'COM7'  # cambia según tu sistema
    velocidad = 115200

    ser = serial.Serial(puerto, velocidad)
    time.sleep(2)

    print("Recibiendo datos del micro:bit...\n")

    try:
        while True:
            linea = ser.readline().decode('utf-8').strip()
            print(linea)
    except KeyboardInterrupt:
        ser.close()
        print("Conexión cerrada.")

<img width="897" height="920" alt="image" src="https://github.com/user-attachments/assets/22704cee-e2a9-4236-9ab9-959594dd840f" />

---

### 🦀 Rust

```rust
use std::io::{BufRead, BufReader};
use std::time::Duration;
use std::thread::sleep;
use serialport::prelude::*;

fn main() {
    let port_name = "COM7";
    let baud_rate = 115200;

    let settings = SerialPortSettings {
        baud_rate,
        timeout: Duration::from_millis(1000),
        ..Default::default()
    };

    match serialport::open_with_settings(port_name, &settings) {
        Ok(port) => {
            let mut reader = BufReader::new(port);
            println!("Recibiendo datos del micro:bit...\n");
            sleep(Duration::from_secs(2));

            for line in reader.lines() {
                match line {
                    Ok(data) => println!("{}", data),
                    Err(e) => eprintln!("Error leyendo: {}", e),
                }
            }
        }
        Err(e) => eprintln!("No se pudo abrir el puerto: {}", e),
    }
}
```

---

### 💠 C++

```cpp
#include <iostream>
#include <fstream>
#include <string>
#include <thread>
#include <chrono>
#include <windows.h>

int main() {
    const char* port = "\\\\.\\COM7";
    HANDLE hSerial = CreateFile(port, GENERIC_READ, 0, NULL, OPEN_EXISTING, 0, NULL);

    if (hSerial == INVALID_HANDLE_VALUE) {
        std::cerr << "No se pudo abrir el puerto.\n";
        return 1;
    }

    DCB dcbSerialParams = {0};
    dcbSerialParams.DCBlength = sizeof(dcbSerialParams);
    GetCommState(hSerial, &dcbSerialParams);
    dcbSerialParams.BaudRate = CBR_115200;
    dcbSerialParams.ByteSize = 8;
    dcbSerialParams.StopBits = ONESTOPBIT;
    dcbSerialParams.Parity   = NOPARITY;
    SetCommState(hSerial, &dcbSerialParams);

    std::this_thread::sleep_for(std::chrono::seconds(2));
    std::cout << "Recibiendo datos del micro:bit...\n\n";

    char buffer[256];
    DWORD bytesRead;
    while (true) {
        if (ReadFile(hSerial, buffer, sizeof(buffer) - 1, &bytesRead, NULL)) {
            buffer[bytesRead] = '\0';
            std::cout << buffer;
        }
    }

    CloseHandle(hSerial);
    return 0;
}
```

---

### 🌀 Go

```go
package main

import (
    "bufio"
    "fmt"
    "go.bug.st/serial"
    "time"
)

func main() {
    portName := "COM7"
    mode := &serial.Mode{
        BaudRate: 115200,
    }

    port, err := serial.Open(portName, mode)
    if err != nil {
        fmt.Println("No se pudo abrir el puerto:", err)
        return
    }
    defer port.Close()

    time.Sleep(2 * time.Second)
    fmt.Println("Recibiendo datos del micro:bit...\n")

    scanner := bufio.NewScanner(port)
    for scanner.Scan() {
        fmt.Println(scanner.Text())
    }

    if err := scanner.Err(); err != nil {
        fmt.Println("Error leyendo:", err)
    }
}
```

---

### 🧩 C#

```csharp
using System;
using System.IO.Ports;
using System.Threading;

class Program
{
    static void Main()
    {
        string portName = "COM7";
        int baudRate = 115200;

        SerialPort serialPort = new SerialPort(portName, baudRate);
        serialPort.Open();
        Thread.Sleep(2000);

        Console.WriteLine("Recibiendo datos del micro:bit...\n");

        try
        {
            while (true)
            {
                string line = serialPort.ReadLine();
                Console.WriteLine(line.Trim());
            }
        }
        catch (Exception ex)
        {
            Console.WriteLine("Error: " + ex.Message);
        }
        finally
        {
            serialPort.Close();
            Console.WriteLine("Conexión cerrada.");
        }
    }
}
```

