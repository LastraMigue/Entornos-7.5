# Entornos-7.5 - GymMaster

## Fase 1: Análisis de Requisitos (Criterios a, b)
El sistema debe permitir que los **Socios** se identifiquen y reserven clases (como Yoga o Crossfit). Si una clase está llena, el socio puede apuntarse a una **Lista de Espera**. El **Administrador** debe poder dar de alta nuevas clases y cancelar sesiones si el monitor no asiste.

**Tarea 1:** Elabora el **Diagrama de Casos de Uso**.

* Identifica al menos 2 actores.
* Incluye relaciones &lt;&lt;include&gt;&gt; (ej. para el login) y &lt;&lt;extend&gt;&gt; (ej. para la lista de espera).
* Define el límite del sistema.

```mermaid
graph LR

%% Actores
Member([Member])
Admin([Administrator])

%% Límite del Sistema y Casos de Uso
subgraph "Gym Reservation System"
UC_Login(Login)
UC_BookClass(Book Class)
UC_JoinWaitlist(Join Waitlist)
UC_CreateClass(Create Class)
UC_CancelSession(Cancel Session)
end

%% Relaciones actor-casos de uso
Member --> UC_BookClass
Admin --> UC_CreateClass
Admin --> UC_CancelSession

%% Relaciones <<include>>
UC_BookClass -.-> |&lt;&lt;include&gt;&gt;| UC_Login
UC_CreateClass -.-> |&lt;&lt;include&gt;&gt;| UC_Login
UC_CancelSession -.-> |&lt;&lt;include&gt;&gt;| UC_Login

%% Relaciones <<extend>>
UC_JoinWaitlist -.-> |&lt;&lt;extend&gt;&gt;| UC_BookClass
```

---

## Fase 2: Diseño de la Interacción (Criterios c, d)
Nos centramos en el momento exacto en que un Socio pulsa el botón "Confirmar Reserva".

**Tarea 2:** Elabora un **Diagrama de Secuencia** para el proceso Confirmar Reserva.

* **Objetos involucrados:** :Socio, :InterfazWeb, :GestorReservas, :BaseDatos.
* **Flujo:** El socio envía la petición; el gestor comprueba disponibilidad en la BD; la BD responde; el gestor confirma y la interfaz muestra el mensaje de éxito.
* **Nota:** Usa fragmentos combinados (alt o opt) para gestionar qué pasa si no hay hueco.

```mermaid
sequenceDiagram
autonumber

%% Actores y Objetos
actor Member
participant WebInterface
participant ReservationManager
participant Database

Note over Member, Database: BOOKING CONFIRMATION FLOW

%% El Socio inicia la acción y activa la Interfaz Web
Member->>WebInterface: confirmReservation()
activate WebInterface

%% La interfaz llama al gestor y lo activa
WebInterface->>ReservationManager: processReservation(memberId, classId)
activate ReservationManager

%% El gestor consulta la BD y la activa
ReservationManager->>Database: checkAvailability(classId)
activate Database
    
Note right of Database: We check if there are any vacancies

%% La BD responde y se desactiva a sí misma
Database-->>ReservationManager: availabilityStatus
deactivate Database

%% Fragmento combinado (Alternativa lógica)
alt isAvailable == true
    %% Camino A: Hay hueco
    ReservationManager->>Database: blockSpot(memberId, classId)
    activate Database
        
    Database-->>ReservationManager: successConfirmation
    deactivate Database

    %% El gestor responde a la interfaz
    ReservationManager-->>WebInterface: notifySuccess()

    %% La interfaz responde al socio
    WebInterface-->>Member: showSuccessMessage()

else isAvailable == false
    %% Camino B: Clase llena
    %% El gestor avisa a la interfaz
    ReservationManager-->>WebInterface: notifyFullCapacity()
    deactivate ReservationManager

    %% La interfaz avisa al socio
    WebInterface-->>Member: showWaitlistOption()
    deactivate WebInterface
end
```

**Tarea 3:** Elabora un **Diagrama de Comunicación** equivalente al anterior.

* Muestra la misma interacción pero enfocada en los enlaces entre objetos.
* Utiliza correctamente la numeración decimal (1, 1.1, 2...) para el orden de los mensajes.

```mermaid
graph LR

%% Definición de los objetos
Member((:Member))
WebInterface((:WebInterface))
ReservationManager((:ReservationManager))
Database((:Database))

%% --- FLUJO PRINCIPAL ---
Member
-- "1: confirmReservation()"
--> WebInterface

WebInterface
-- "1.1: processReservation(memberId, classId)"
--> ReservationManager

ReservationManager
-- "1.1.1: checkAvailability(classId)"
--> Database

Database
-- "1.1.2: availabilityStatus"
--> ReservationManager


%% --- ALTERNATIVA A: Hay hueco (isAvailable=true) ---
ReservationManager
-- "1.1.3a: [isAvailable=true] blockSpot(memberId, classId)"
--> Database

Database
-- "1.1.4a: [isAvailable=true] successConfirmation"
--> ReservationManager

ReservationManager
-- "1.1.5a: [isAvailable=true] notifySuccess()"
--> WebInterface

WebInterface
-- "1.1.6a: [isAvailable=true] showSuccessMessage()"
--> Member


%% --- ALTERNATIVA B: Clase llena (isAvailable=false) ---
ReservationManager
-- "1.1.3b: [isAvailable=false] notifyFullCapacity()"
--> WebInterface

WebInterface
-- "1.1.4b: [isAvailable=false] showWaitlistOption()"
--> Member
```

---

## Fase 3: Lógica del Proceso (Criterios e, f)
Antes de confirmar la reserva, el gimnasio sigue un protocolo interno de seguridad y pagos.

**Tarea 4:** Elabora un **Diagrama de Actividades** para el flujo Validación de Reserva.

* **Pasos:** 1. Recibir solicitud -> 2. ¿Socio tiene cuota pagada? (Decisión) -> 3. ¿Hay aforo? (Decisión) -> 4. Bloquear plaza -> 5. Enviar email de confirmación.
* Usa correctamente los símbolos de inicio, fin, acciones y rombos de decisión.

```mermaid
stateDiagram-v2
    
%% Decisiones
state if_fee <<choice>>
state if_capacity <<choice>>

%% Flujo
[*] --> ReceiveRequest
ReceiveRequest --> if_fee
    
%% Primera Decisión: Cuota
if_fee --> if_capacity : [fee paid]
if_fee --> RejectUnpaid : [unpaid fee]
    
%% Segunda Decisión: Aforo
if_capacity --> BlockSpot : [capacity]
if_capacity --> RejectFull : [no capacity]
    
%% Finalización exitosa
BlockSpot --> SendConfirmationEmail
SendConfirmationEmail --> [*]
```
