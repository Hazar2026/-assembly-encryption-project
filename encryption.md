section .data
    msgPrompt db "Enter plaintext: "
    msgPromptLen equ $ - msgPrompt

    keyPrompt db "Enter key: "
    keyPromptLen equ $ - keyPrompt

    plainLabel db "Plain text: "
    plainLabelLen equ $ - plainLabel

    keyLabel db "Key: "
    keyLabelLen equ $ - keyLabel

    encLabel db "Encrypted text: "
    encLabelLen equ $ - encLabel

    decLabel db "Decrypted text: "
    decLabelLen equ $ - decLabel

    filename db "output.txt", 0
    newline db 10

section .bss
    message resb 50
    key resb 50
    encrypted resb 50
    decrypted resb 50
    length resd 1
    fd resd 1

section .text
global _start

_start:

    ; Ask for plaintext
    mov eax, 4
    mov ebx, 1
    mov ecx, msgPrompt
    mov edx, msgPromptLen
    int 0x80

    ; Read plaintext
    mov eax, 3
    mov ebx, 0
    mov ecx, message
    mov edx, 50
    int 0x80

    dec eax
    mov [length], eax

    ; Ask for key
    mov eax, 4
    mov ebx, 1
    mov ecx, keyPrompt
    mov edx, keyPromptLen
    int 0x80

    ; Read key
    mov eax, 3
    mov ebx, 0
    mov ecx, key
    mov edx, 50
    int 0x80

    ; Encrypt
    xor esi, esi

encrypt:
    cmp esi, [length]
    je start_decrypt

    mov al, [message + esi]
    xor al, [key + esi]
    mov [encrypted + esi], al

    inc esi
    jmp encrypt

start_decrypt:
    xor esi, esi

decrypt:
    cmp esi, [length]
    je create_file

    mov al, [encrypted + esi]
    xor al, [key + esi]
    mov [decrypted + esi], al

    inc esi
    jmp decrypt

create_file:

    mov eax, 8
    mov ebx, filename
    mov ecx, 0644o
    int 0x80
    mov [fd], eax

    ; Plain text
    mov eax, 4
    mov ebx, [fd]
    mov ecx, plainLabel
    mov edx, plainLabelLen
    int 0x80

    mov eax, 4
    mov ebx, [fd]
    mov ecx, message
    mov edx, [length]
    int 0x80

    call new_line

    ; Key
    mov eax, 4
    mov ebx, [fd]
    mov ecx, keyLabel
    mov edx, keyLabelLen
    int 0x80

    mov eax, 4
    mov ebx, [fd]
    mov ecx, key
    mov edx, [length]
    int 0x80

    call new_line

    ; Encrypted text
    mov eax, 4
    mov ebx, [fd]
    mov ecx, encLabel
    mov edx, encLabelLen
    int 0x80

    mov eax, 4
    mov ebx, [fd]
    mov ecx, encrypted
    mov edx, [length]
    int 0x80

    call new_line

    ; Decrypted text
    mov eax, 4
    mov ebx, [fd]
    mov ecx, decLabel
    mov edx, decLabelLen
    int 0x80

    mov eax, 4
    mov ebx, [fd]
    mov ecx, decrypted
    mov edx, [length]
    int 0x80

    call new_line

    ; Close file
    mov eax, 6
    mov ebx, [fd]
    int 0x80

    ; Exit
    mov eax, 1
    mov ebx, 0
    int 0x80


new_line:
    mov eax, 4
    mov ebx, [fd]
    mov ecx, newline
    mov edx, 1
    int 0x80
    ret