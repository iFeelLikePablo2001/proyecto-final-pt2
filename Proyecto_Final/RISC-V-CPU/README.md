# Procesador RISC-V Uniciclo

**Diseño de Sistemas Digitales (EL-3310) · Instituto Tecnológico de Costa Rica**
**Profesor:** Ing. Michael González Vega · **Grupo 4: Los Ohmnívoros**

Procesador RISC-V de 32 bits, uniciclo, en SystemVerilog. Ejecuta `add`, `sub`, `and`, `or`, `addi`, `lw` y `sw` con ejecución secuencial (sin saltos de direccion).

| Capítulo | Contenido | Estado |
|---|---|---|
| 1 · Los cimientos | ALU, Register File, Immediate Generator, MUX y sus testbenches | Entregado y verificado |
| 2 · El datapath cobra vida | Program Counter, memorias, Control Unit, Processor y testbench de integración | Entregado y verificado |

---

## Integrantes y roles

| Integrante | Rol | Aporte principal |
|---|---|---|
| Pablo Andrés Cabrera Montealegre | Hardware Developer 1 | `imm_gen`, `mux`, `control_unit`, `processor` y documentacion|
| Julián Murillo Wo Ching | Hardware Developer 2 | `alu`, `register_file`, `program_counter`, `instruction_memory`, `data_memory` |
| Iván Josué Arce Quirós | Verification Engineer | Todos los testbenches y casos de prueba |

Los roles del Capítulo 1 se mantienen en el Capítulo 2.

---

## Estructura del repositorio

```text
RISC-V-CPU/
├── src/                                  Módulos SystemVerilog
│   ├── alu.sv                            (Cap. 1)
│   ├── register_file.sv                  (Cap. 1)
│   ├── imm_gen.sv                        (Cap. 1)
│   ├── mux.sv                            (Cap. 1)
│   ├── program_counter.sv                (Cap. 2)
│   ├── instruction_memory.sv             (Cap. 2)
│   ├── data_memory.sv                    (Cap. 2)
│   ├── control_unit.sv                   (Cap. 2)
│   └── processor.sv                      (Cap. 2)
│
├── testbench/                            Un testbench por módulo
│   ├── alu_tb.sv
│   ├── register_file_tb.sv
│   ├── imm_gen_tb.sv
│   ├── mux_tb.sv
│   ├── program_counter_tb.sv
│   ├── instruction_memory_tb.sv
│   ├── data_memory_tb.sv
│   ├── control_unit_tb.sv
│   ├── processor_tb.sv                   Integracion final
│   ├── Links/                            Enlaces de EDA Playground
│   └── Imagenes de Pruebas/              Evidencia: capturas de las simulaciones, por módulo
│
├── docs/                                 Informes técnicos
│   ├── Grupo4_Informe_Capitulo1.pdf
│   └── Grupo4_Informe_Capitulo2.pdf      
│
└── README.md
```

---

## Enlaces de simulación (EDA Playground)

Cada enlace abre un Playground con el módulo y su testbench listos para ejecutar. Los mismos enlaces están en `testbench/Links/`.

| Simulación | Enlace |
|---|---|
| **Processor** | https://www.edaplayground.com/x/f3wF |
| ALU | https://www.edaplayground.com/x/rArT |
| Register File | https://www.edaplayground.com/x/9NTD |
| Immediate Generator | https://www.edaplayground.com/x/8Eq_ |
| MUX | https://www.edaplayground.com/x/vY9z |
| Program Counter | https://www.edaplayground.com/x/ZXzW |
| Instruction Memory | https://www.edaplayground.com/x/7cbz |
| Data Memory | https://www.edaplayground.com/x/QuXL |
| Control Unit | https://www.edaplayground.com/x/w4aB |

---

## Estado de los módulos

| Módulo | Archivo | Implementación | Verificación | Casos | Estado |
|---|---|---|---|---:|---|
| ALU | `alu.sv` | `always_comb` con `case` sobre `ALUControl`; `Zero` se deriva del resultado | `alu_tb.sv`: ADD, SUB, AND, OR, cero, negativo y desbordamiento | 8 | Verificado |
| Register File | `register_file.sv` | 32×32 bits, lectura combinacional, escritura síncrona con `RegWrite`, `x0` fijo en 0 | `register_file_tb.sv`: escritura/lectura, lectura doble, protección de `x0`, `RegWrite=0` | 4 | Verificado |
| Immediate Generator | `imm_gen.sv` | Extensión de signo I-type (`ImmSrc=00`) y S-type (`ImmSrc=01`) | `imm_gen_tb.sv`: positivos, negativos, máximo y mínimo de ambos formatos | 8 | Verificado |
| MUX 2:1 | `mux.sv` (`mux2to1`) | Parametrizable con `WIDTH` (32 por defecto) | `mux_tb.sv`: ambas entradas con tres pares de datos | 6 | Verificado |
| Program Counter | `program_counter.sv` | Registro de 32 bits, reset síncrono activo en alto a `0x00000000` | `program_counter_tb.sv`: reset, cargas y prioridad del reset | 6 | Verificado |
| Instruction Memory | `instruction_memory.sv` | 64 palabras, lectura combinacional, relleno con NOP, índice `A[7:2]` | `instruction_memory_tb.sv`: inicialización, lecturas y última posición | 6 | Verificado |
| Data Memory | `data_memory.sv` | 64 palabras, lectura combinacional, escritura síncrona con `MemWrite` | `data_memory_tb.sv`: escritura, conservación, `MemWrite=0` y temporización | 8 | Verificado |
| Control Unit | `control_unit.sv` | Decodifica `opcode`/`funct3`/`funct7`; valores por defecto sin escritura | `control_unit_tb.sv`: 7 instrucciones y 3 casos no soportados | 10 | Verificado |
| Processor | `processor.sv` | Integra PC, memorias, control, banco de registros, `imm_gen`, 2 MUX y ALU | `processor_tb.sv`: programa de 12 instrucciones, PC y valores de registros y memoria | 27 | Verificado |
| **Total** | — | — | — | **83** | **83/83 PASS** |

---

## Diseño

### Instrucciones soportadas

| Instrucción | Formato | Operación |
|---|:-:|---|
| `add rd, rs1, rs2` | R | `rd = rs1 + rs2` |
| `sub rd, rs1, rs2` | R | `rd = rs1 - rs2` |
| `and rd, rs1, rs2` | R | `rd = rs1 & rs2` |
| `or rd, rs1, rs2` | R | `rd = rs1 \| rs2` |
| `addi rd, rs1, imm` | I | `rd = rs1 + imm` |
| `lw rd, imm(rs1)` | I | `rd = Mem[rs1 + imm]` |
| `sw rs2, imm(rs1)` | S | `Mem[rs1 + imm] = rs2` |

### Señales de control

| Instrucción | opcode | funct3 | funct7 | RegWrite | ALUSrc | ALUControl | ImmSrc | MemWrite | ResultSrc |
|---|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|
| `add` | 0110011 | 000 | 0000000 | 1 | 0 | 0000 | 00 | 0 | 0 |
| `sub` | 0110011 | 000 | 0100000 | 1 | 0 | 0001 | 00 | 0 | 0 |
| `and` | 0110011 | 111 | 0000000 | 1 | 0 | 0010 | 00 | 0 | 0 |
| `or` | 0110011 | 110 | 0000000 | 1 | 0 | 0011 | 00 | 0 | 0 |
| `addi` | 0010011 | 000 | no importa | 1 | 1 | 0000 | 00 | 0 | 0 |
| `lw` | 0000011 | 010 | no importa | 1 | 1 | 0000 | 00 | 0 | 1 |
| `sw` | 0100011 | 010 | no importa | 0 | 1 | 0000 | 01 | 1 | 0 |
| No soportada | otro | otro | otro | 0 | 0 | 0000 | 00 | 0 | 0 |

Para el caso de una instruccion fuera de las especificadas (con un opcode, `funct3` o `funct7` fuera de la tabla) las senales permanecen en su default, sin escribir en registros ni memoria.

### Cambios realizados segun observaciones del profesor

| Módulo | Cambio | Fecha | Responsable |
|---|---|---|---|
| `mux.sv` | Los puertos `A`, `B` y `Y` pasan de 32 bits fijos a `[WIDTH-1:0]`, de modo que el parámetro `WIDTH` tenga efecto real | 06/11/2026 | Julián Murillo |
| `register_file.sv`, `alu.sv` | Se llena la informacion de la descripcion; sin cambio funcional | 10/11/2026 | Julián Murillo |
| `imm_gen.sv` | Se llena el encabezado | 20/10/2026 | Pablo Cabrera |

---

## Programa de prueba utilizado en `processor_tb.sv`

El testbench carga 12 instrucciones ejecutables y las compara con los resultados esperados descritos en la siguiente tabla:

| Dirección | Código | Instrucción | Resultado esperado |
|:-:|:-:|---|---|
| 0x00 | `0x00D00093` | `addi x1, x0, 13` | x1 = 13 |
| 0x04 | `0x00700113` | `addi x2, x0, 7` | x2 = 7 |
| 0x08 | `0x002081B3` | `add x3, x1, x2` | x3 = 20 |
| 0x0C | `0x40208233` | `sub x4, x1, x2` | x4 = 6 |
| 0x10 | `0x0020F2B3` | `and x5, x1, x2` | x5 = 5 |
| 0x14 | `0x0020E333` | `or x6, x1, x2` | x6 = 15 |
| 0x18 | `0x00402023` | `sw x4, 0(x0)` | Mem[0] = 6 |
| 0x1C | `0x00002383` | `lw x7, 0(x0)` | x7 = 6 |
| 0x20 | `0xFFD38413` | `addi x8, x7, -3` | x8 = 3 |
| 0x24 | `0x00802223` | `sw x8, 4(x0)` | Mem[4] = 3 |
| 0x28 | `0x00402483` | `lw x9, 4(x0)` | x9 = 3 |
| 0x2C | `0x00548533` | `add x10, x9, x5` | x10 = 8 |

Además se comprueba el reset inicial PC = 0 y que `x0` permanece en cero. 

---

## Evidencia de verificación

- **Simulaciones:** capturas de cada módulo en `testbench/Imagenes de Pruebas/<Módulo>/`, donde cada caso muestra entradas, resultado esperado, resultado obtenido y `[PASS]`.
- **Informe técnico del Capítulo 2:** `docs/Grupo4_Informe_Capitulo2.pdf`, con el diagrama del datapath, la tabla de control, la estrategia de verificación y la matriz de pruebas completa (83 casos).
- **Informe técnico del Capítulo 1:** `docs/Grupo4_Informe_Capitulo1.pdf`.
