# ShellcodeLoad-Golang

Este es un cargador de shellcode de Windows de alto nivel desarrollado en Go, que integra diversas técnicas modernas de evasión y contramedidas de equipos rojos. Este proyecto está destinado exclusivamente a la investigación de seguridad legítima y a las pruebas de penetración autorizadas.

## Características principales

- **Llamadas al sistema directas:** Implementadas en lenguaje ensamblador, omitiendo los ganchos en línea de la API en modo usuario.
- **Suplantación de pila:** Simula una pila de llamadas legítima para evadir la detección basada en el análisis de la pila de llamadas.
- **Conmutación de protección de memoria (RW -> RX):** Emplea un esquema de asignación de memoria en dos etapas para evitar la asignación de memoria RWX obvia.
- **Ejecución de fibra:** Utiliza el mecanismo de Windows Fiber para ejecutar cargas útiles en modo usuario, proporcionando contexto independiente y capacidades de evasión.
- **Suspensión inteligente/Enmascaramiento de suspensión:** Cifra y modifica los atributos de memoria (RW) durante la suspensión, evadiendo el escaneo de memoria.
- **Resolución dinámica de SSN:** Obtiene dinámicamente los números de llamada del sistema (SSN) en tiempo de ejecución, sin depender de la codificación.
- **Antidepuración y antisandboxing:** Integra múltiples lógicas de detección de entornos para identificar depuradores, máquinas virtuales y entornos sandbox.
- **Descifrado de carga útil**: Admite la adquisición remota y el descifrado en tiempo real de cargas útiles cifradas de ChaCha20.

## Inicio rápido

1. **Preparación del entorno**: Instale el entorno Go (se recomienda la versión 1.20 o superior) y configure el entorno de compilación cruzada de Windows.
2. **Configuración de URL**: Modifique `configURL` en `main.go` con la dirección de su archivo de configuración de C2.
3. **Compilación**:   
   ```bash
   GOOS=windows GOARCH=amd64 go build -ldflags="-s -w -H=windowsgui" -o loader.exe
   ```
## Gráfico de detección de VT:
![photo_6337066581753532155_w](https://github.com/user-attachments/assets/955229d0-bf55-4ed1-b153-014529ddfe31)

## Prueba del funcionamiento sin movimiento del 360
<img width="1355" height="829" alt="image" src="https://github.com/user-attachments/assets/890cdfeb-945a-4d88-87c3-64c7abb351b3" />

## Manual del usuario

https://github.com/itboxltda/shellcodeload_golang/blob/main/%E4%BD%BF%E7%94%A8%E6%89%8B%E5%86%8C.md

## Documentación

https://github.com/itboxltda/shellcodeload_golang/tree/main#:~:text=6%20hours%20ago-,%E8%AF%B4%E6%98%8E%E6%96%87%E6%A1%A3.md,-Initial%20commit%3A%20Advanced

## Descargo de responsabilidad

Esta herramienta es solo para fines de investigación y aprendizaje en seguridad. Los usuarios deben cumplir con las leyes y normativas locales y tienen estrictamente prohibido usarla con fines ilegales. El autor no se responsabiliza de las consecuencias derivadas del uso de esta herramienta.
