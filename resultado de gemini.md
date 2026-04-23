¡Hola! Como desarrollador de software, me encanta la estructura que planteas. Vamos a construir este proyecto **PerfumeríaCRUD** con un enfoque profesional, integrando Firebase y siguiendo la metodología de **Antigravity** (que se centra en la agilidad y el flujo de trabajo estructurado por agentes/roles).

Aquí tienes el plan de batalla detallado:

---

## 1. Configuración Inicial de Entorno
Primero, preparamos el espacio de trabajo en tu terminal o gestor de archivos:

* **Ruta:** `xflutterdominguez0566/perfumeriacrud`
* **Comando para crear el proyecto:**
    ```bash
    mkdir xflutterdominguez0566
    cd xflutterdominguez0566
    flutter create perfumeriacrud
    cd perfumeriacrud
    ```

---

## 2. Configuración de Firebase Console (Base de Datos)
Para la colección de **empleados**, sigue estos pasos en la [Consola de Firebase](https://console.firebase.google.com/):

1.  Crea un nuevo proyecto llamado `PerfumeriaCRUD`.
2.  En el menú lateral, ve a **Firestore Database** y haz clic en **Crear base de datos**.
3.  Selecciona **Modo de prueba** (para permitir lectura/escritura inicial sin restricciones de seguridad complejas).
4.  Crea una colección llamada `empleados`.
5.  Agrega un documento inicial con los campos: `nombre` (string), `apellido` (string) y `salario` (number).

---

## 3. Integración de Librerías y `pubspec.yaml`
Para que Flutter hable con Firebase, necesitamos las dependencias oficiales. Modifica tu archivo `pubspec.yaml` en la sección de `dependencies`:

```yaml
dependencies:
  flutter:
    sdk: flutter
  # Librerías Core e integraciones
  firebase_core: ^2.24.2
  cloud_firestore: ^4.14.0
  cupertino_icons: ^1.0.2
```

> **Nota:** Después de guardar, ejecuta `flutter pub get` en la terminal.

---

## 4. Metodología Antigravity: Agentes y Flujo
Bajo la filosofía Antigravity, dividimos el desarrollo en **Agentes de Responsabilidad**:

| Agente | Rol | Skill (Habilidad) |
| :--- | :--- | :--- |
| **Architect** | Definir estructura de datos | Diseño de Modelos Dart |
| **Firebase Handler** | Comunicación con Firestore | CRUD Ops (Streams/Futures) |
| **UI Designer** | Interfaz de usuario | Widgets con colores atractivos |

### Estructura de Carpetas (Clean Architecture)
```text
lib/
├── models/          # Agente Architect (Modelo de datos)
├── services/        # Agente Firebase Handler (Lógica Firestore)
├── screens/         # Agente UI Designer (Vistas)
└── main.dart        # Punto de entrada
```

---

## 5. Implementación del Código (Totalmente Funcional)

### A. El Modelo (`lib/models/empleado_model.dart`)
```dart
class Empleado {
  String id;
  String nombre;
  String apellido;
  double salario;

  Empleado({required this.id, required this.nombre, required this.apellido, required this.salario});

  // Convertir de Firestore a Objeto
  factory Empleado.fromMap(Map<String, dynamic> map, String id) {
    return Empleado(
      id: id,
      nombre: map['nombre'] ?? '',
      apellido: map['apellido'] ?? '',
      salario: (map['salario'] ?? 0).toDouble(),
    );
  }

  // Convertir de Objeto a Firestore
  Map<String, dynamic> toMap() {
    return {
      'nombre': nombre,
      'apellido': apellido,
      'salario': salario,
    };
  }
}
```

### B. El Servicio CRUD (`lib/services/firebase_service.dart`)
```dart
import 'cloud_firestore/cloud_firestore.dart';
import '../models/empleado_model.dart';

class FirebaseService {
  final CollectionReference _db = FirebaseFirestore.instance.collection('empleados');

  // LEER (Stream para tiempo real)
  Stream<List<Empleado>> getEmpleados() {
    return _db.snapshots().map((snapshot) =>
        snapshot.docs.map((doc) => Empleado.fromMap(doc.data() as Map<String, dynamic>, doc.id)).toList());
  }

  // CREAR
  Future<void> addEmpleado(Empleado emp) => _db.add(emp.toMap());

  // ACTUALIZAR
  Future<void> updateEmpleado(Empleado emp) => _db.doc(emp.id).update(emp.toMap());

  // BORRAR
  Future<void> deleteEmpleado(String id) => _db.doc(id).delete();
}
```

### C. La Interfaz de Usuario (`lib/screens/home_screen.dart`)
He seleccionado una paleta de colores **Deep Purple y Amber** para que sea visualmente atractiva.

```dart
import 'package:flutter/material.dart';
import '../services/firebase_service.dart';
import '../models/empleado_model.dart';

class HomeScreen extends StatelessWidget {
  final FirebaseService _service = FirebaseService();

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text('Perfumería CRUD - Empleados'),
        backgroundColor: Colors.deepPurple,
        foregroundColor: Colors.white,
      ),
      body: StreamBuilder<List<Empleado>>(
        stream: _service.getEmpleados(),
        builder: (context, snapshot) {
          if (!snapshot.hasData) return Center(child: CircularProgressIndicator());
          
          final empleados = snapshot.data!;
          return ListView.builder(
            itemCount: empleados.length,
            itemBuilder: (context, index) {
              final emp = empleados[index];
              return Card(
                elevation: 4,
                margin: EdgeInsets.symmetric(horizontal: 10, vertical: 5),
                child: ListTile(
                  leading: CircleAvatar(backgroundColor: Colors.amber, child: Text(emp.nombre[0])),
                  title: Text('${emp.nombre} ${emp.apellido}', style: TextStyle(fontWeight: FontWeight.bold)),
                  subtitle: Text('Salario: \$${emp.salario}', style: TextStyle(color: Colors.green[700])),
                  trailing: Row(
                    mainAxisSize: MainAxisSize.min,
                    children: [
                      IconButton(icon: Icon(Icons.edit, color: Colors.blue), onPressed: () => _showForm(context, emp)),
                      IconButton(icon: Icon(Icons.delete, color: Colors.red), onPressed: () => _service.deleteEmpleado(emp.id)),
                    ],
                  ),
                ),
              );
            },
          );
        },
      ),
      floatingActionButton: FloatingActionButton(
        backgroundColor: Colors.deepPurple,
        child: Icon(Icons.add, color: Colors.white),
        onPressed: () => _showForm(context, null),
      ),
    );
  }

  void _showForm(BuildContext context, Empleado? empleado) {
    final nomController = TextEditingController(text: empleado?.nombre);
    final apeController = TextEditingController(text: empleado?.apellido);
    final salController = TextEditingController(text: empleado?.salario.toString());

    showModalBottomSheet(
      isScrollControlled: true,
      context: context,
      builder: (_) => Padding(
        padding: EdgeInsets.only(bottom: MediaQuery.of(context).viewInsets.bottom, left: 15, right: 15, top: 15),
        child: Column(
          mainAxisSize: MainAxisSize.min,
          children: [
            TextField(controller: nomController, decoration: InputDecoration(labelText: 'Nombre')),
            TextField(controller: apeController, decoration: InputDecoration(labelText: 'Apellido')),
            TextField(controller: salController, decoration: InputDecoration(labelText: 'Salario'), keyboardType: TextInputType.number),
            SizedBox(height: 20),
            ElevatedButton(
              style: ElevatedButton.styleFrom(backgroundColor: Colors.deepPurple),
              child: Text(empleado == null ? 'Crear' : 'Actualizar', style: TextStyle(color: Colors.white)),
              onPressed: () {
                final emp = Empleado(
                  id: empleado?.id ?? '',
                  nombre: nomController.text,
                  apellido: apeController.text,
                  salario: double.parse(salController.text),
                );
                empleado == null ? _service.addEmpleado(emp) : _service.updateEmpleado(emp);
                Navigator.pop(context);
              },
            )
          ],
        ),
      ),
    );
  }
}
```

### D. Archivo Principal (`lib/main.dart`)
```dart
import 'package:flutter/material.dart';
import 'package:firebase_core/firebase_core.dart';
import 'screens/home_screen.dart';

void main() async {
  WidgetsFlutterBinding.ensureInitialized();
  await Firebase.initializeApp(); // Inicialización obligatoria
  runApp(PerfumeriaApp());
}

class PerfumeriaApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      debugShowCheckedModeBanner: false,
      title: 'Perfumería CRUD',
      theme: ThemeData(primarySwatch: Colors.deepPurple, useMaterial3: true),
      home: HomeScreen(),
    );
  }
}
```

---

## 6. Práctica Guiada para Estudiantes (Metodología Antigravity)

Para que tus estudiantes dominen esto, diles que sigan el flujo del **"Círculo de Creación"**:

1.  **Fase de Identidad (Roles):** Divide al grupo. Uno será el "Data Master" (Firestore), otro el "Logic Coder" (Servicios Dart) y otro el "Visual Artist" (UI).
2.  **Sincronización:** El Data Master debe entregar la estructura de la consola antes de que el Logic Coder escriba una sola línea.
3.  **Implementación Incremental:**
    * **Paso 1:** Solo lectura. Si pueden ver los datos de Firebase en la app, la conexión es exitosa.
    * **Paso 2:** Escritura. Implementar el botón "Agregar".
    * **Paso 3:** Edición y Borrado.

¡Con este plan, el proyecto estará funcionando en tiempo récord y con una estructura limpia! ¿Te gustaría que profundice en algún widget específico para mejorar el diseño?
