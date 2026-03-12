# Entornos-7.5 - GymMaster

## Fase 1: Análisis de Requisitos (Criterios a, b)
El sistema debe permitir que los **Socios** se identifiquen y reserven clases (como Yoga o Crossfit). Si una clase está llena, el socio puede apuntarse a una **Lista de Espera**. El **Administrador** debe poder dar de alta nuevas clases y cancelar sesiones si el monitor no asiste.

**Tarea 1:** Elabora el **Diagrama de Casos de Uso**.

* Identifica al menos 2 actores.
* Incluye relaciones <<include>> (ej. para el login) y <<extend>> (ej. para la lista de espera).
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

