# cryptolib（密码）

## class

- class cryptolib.`aes`

  - classmethod `__init__`(key, mode[, IV])
    
    初始化密码对象，适用于加密 / 解密操作。注意：初始化后，密码对象仅能用于加密或解密其中一种操作，不支持在加密后执行解密操作，反之亦然。
  
    参数说明：
    - `key`：加密 / 解密密钥（类字节对象）。
    - `mode`：
      - 1（或 cryptolib.MODE_ECB）：电子密码本模式（ECB）。
      - 2（或 cryptolib.MODE_CBC）：密码块链接模式（CBC）。
      - 6（或 cryptolib.MODE_CTR）：计数器模式（CTR）。
    - `IV`：CBC 模式下的初始化向量；CTR 模式下为计数器初始值。
<br><br>

  - `encrypt`(in_buf[, out_buf])
  
    对 `in_buf` 进行加密。若未指定 `out_buf`，结果将作为新分配的字节对象返回；否则，结果将写入可变缓冲区 `out_buf`。`in_buf` 和 `out_buf` 可指向同一缓冲区，此时数据将原地加密。
<br><br>

  - `decrypt`(in_buf[, out_buf])
  
    功能与 `encrypt()` 类似，但用于解密操作。
