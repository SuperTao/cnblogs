给android的wifi设备添加PMF支持时，抓取omnipeek分析。
```
从assoc req 中发现相关标志位没有使能，说明STA 没有使能PMF
  RSN Capabilities:     %0000000000000000  [74-75]
                        xx...... ........ Reserved
                        ........ 0....... Management Frame Protection Capable (MFPC): disabled   （optional）
                        ........ .0...... Management Frame Protection Required (MFPR): not mandatory  (required）
 
而 Probe Response中使能了相关标志位。说明AP具备PMF 能力
  RSN Capabilities:     %0000000010000000  [94-95]
                        ........ 1....... Management Frame Protection Capable (MFPC): enabled   
                        ........ .0...... Management Frame Protection Required (MFPR): not mandatory

```