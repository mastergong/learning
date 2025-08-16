# Design 05 — Patterns Quick Cheat Sheet

- Creational: Factory Method, Abstract Factory, Builder, Prototype, Singleton
- Structural: Adapter, Bridge, Composite, Decorator, Facade, Flyweight, Proxy
- Behavioral: Chain of Responsibility, Command, Interpreter, Iterator, Mediator, Memento, Observer, State, Strategy, Template Method, Visitor

คำแนะนำเลือกใช้:
- เปลี่ยนพฤติกรรม runtime: Strategy/State
- ครอบ/เสริมความสามารถ: Decorator
- รวมหลาย subsystem: Facade
- เลื่อนสร้าง object ให้ที่อื่น: Factory/Abstract Factory/Builder
- แยกการพึ่งพาระบบเก่า: Adapter/Proxy

สัญญาณเตือน (code smells):
- กิ่ง if-else ตามชนิดมาก: ใช้ Strategy/Polymorphism
- new กระจายในหลายที่: ใช้ Factory/DI Container
- ชั้นที่ขึ้นกับ I/O โดยตรง: ใช้ DIP/Ports-Adapters
