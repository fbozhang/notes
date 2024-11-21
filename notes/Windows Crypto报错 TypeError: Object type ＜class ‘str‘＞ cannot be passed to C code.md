
# 出现的问题

**Windows**使用**Crypto.Cipher.AES**加密需要传递**bytes**类型，而**Linux**传递**str**类型不会报错
# 复现方案

```python
from Crypto.Cipher import AES
AES.new(b'zXP5GtJMmtpw2pIiwCpFcg7L', AES.MODE_CFB, b'zXP5GtJMmtpw2pIi').encrypt("plaintext".encode())
AES.new(b'zXP5GtJMmtpw2pIiwCpFcg7L', AES.MODE_CFB, b'zXP5GtJMmtpw2pIi').encrypt("plaintext")
```
## Windows
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/8e7b4061342bbffb0ad033a0f3989054.png)
## Linux
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/434b51b420a24950525eb31242f1b0df.png)

# 原因
归根到底是因为使用的不是同一个 `AES`类，接下来附上`Windows`和`Linux`的`AES.new`源码即可明白

## Windows
**Windows**的`AES.new`是在`venv/Lib/site-packages/Crypto/Cipher/AES.py`下

```python
def new(key, mode, *args, **kwargs):
    """Create a new AES cipher.

    Args:
      key(bytes/bytearray/memoryview):
        The secret key to use in the symmetric cipher.

        It must be 16 (*AES-128)*, 24 (*AES-192*) or 32 (*AES-256*) bytes long.

        For ``MODE_SIV`` only, it doubles to 32, 48, or 64 bytes.
      mode (a ``MODE_*`` constant):
        The chaining mode to use for encryption or decryption.
        If in doubt, use ``MODE_EAX``.

    Keyword Args:
      iv (bytes/bytearray/memoryview):
        (Only applicable for ``MODE_CBC``, ``MODE_CFB``, ``MODE_OFB``,
        and ``MODE_OPENPGP`` modes).

        The initialization vector to use for encryption or decryption.

        For ``MODE_CBC``, ``MODE_CFB``, and ``MODE_OFB`` it must be 16 bytes long.

        For ``MODE_OPENPGP`` mode only,
        it must be 16 bytes long for encryption
        and 18 bytes for decryption (in the latter case, it is
        actually the *encrypted* IV which was prefixed to the ciphertext).

        If not provided, a random byte string is generated (you must then
        read its value with the :attr:`iv` attribute).

      nonce (bytes/bytearray/memoryview):
        (Only applicable for ``MODE_CCM``, ``MODE_EAX``, ``MODE_GCM``,
        ``MODE_SIV``, ``MODE_OCB``, and ``MODE_CTR``).

        A value that must never be reused for any other encryption done
        with this key (except possibly for ``MODE_SIV``, see below).

        For ``MODE_EAX``, ``MODE_GCM`` and ``MODE_SIV`` there are no
        restrictions on its length (recommended: **16** bytes).

        For ``MODE_CCM``, its length must be in the range **[7..13]**.
        Bear in mind that with CCM there is a trade-off between nonce
        length and maximum message size. Recommendation: **11** bytes.

        For ``MODE_OCB``, its length must be in the range **[1..15]**
        (recommended: **15**).

        For ``MODE_CTR``, its length must be in the range **[0..15]**
        (recommended: **8**).

        For ``MODE_SIV``, the nonce is optional, if it is not specified,
        then no nonce is being used, which renders the encryption
        deterministic.

        If not provided, for modes other than ``MODE_SIV``, a random
        byte string of the recommended length is used (you must then
        read its value with the :attr:`nonce` attribute).

      segment_size (integer):
        (Only ``MODE_CFB``).The number of **bits** the plaintext and ciphertext
        are segmented in. It must be a multiple of 8.
        If not specified, it will be assumed to be 8.

      mac_len (integer):
        (Only ``MODE_EAX``, ``MODE_GCM``, ``MODE_OCB``, ``MODE_CCM``)
        Length of the authentication tag, in bytes.

        It must be even and in the range **[4..16]**.
        The recommended value (and the default, if not specified) is **16**.

      msg_len (integer):
        (Only ``MODE_CCM``). Length of the message to (de)cipher.
        If not specified, ``encrypt`` must be called with the entire message.
        Similarly, ``decrypt`` can only be called once.

      assoc_len (integer):
        (Only ``MODE_CCM``). Length of the associated data.
        If not specified, all associated data is buffered internally,
        which may represent a problem for very large messages.

      initial_value (integer or bytes/bytearray/memoryview):
        (Only ``MODE_CTR``).
        The initial value for the counter. If not present, the cipher will
        start counting from 0. The value is incremented by one for each block.
        The counter number is encoded in big endian mode.

      counter (object):
        (Only ``MODE_CTR``).
        Instance of ``Crypto.Util.Counter``, which allows full customization
        of the counter block. This parameter is incompatible to both ``nonce``
        and ``initial_value``.

      use_aesni: (boolean):
        Use Intel AES-NI hardware extensions (default: use if available).

    Returns:
        an AES object, of the applicable mode.
    """

    kwargs["add_aes_modes"] = True
    return _create_cipher(sys.modules[__name__], key, mode, *args, **kwargs)
```
这个`_create_cipher`其实是调用一个工厂类，忽略相关代码，直接说重点，他实际上是调用`venv/Lib/site-packages/Crypto/Cipher/_mode_cfb.py`下的`CfbMode`类，附上部分源码

```python
class CfbMode(object):
    """*Cipher FeedBack (CFB)*.

    This mode is similar to CFB, but it transforms
    the underlying block cipher into a stream cipher.

    Plaintext and ciphertext are processed in *segments*
    of **s** bits. The mode is therefore sometimes
    labelled **s**-bit CFB.

    An Initialization Vector (*IV*) is required.

    See `NIST SP800-38A`_ , Section 6.3.

    .. _`NIST SP800-38A` : http://csrc.nist.gov/publications/nistpubs/800-38a/sp800-38a.pdf

    :undocumented: __init__
    """

    def __init__(self, block_cipher, iv, segment_size):
        """Create a new block cipher, configured in CFB mode.

        :Parameters:
          block_cipher : C pointer
            A smart pointer to the low-level block cipher instance.

          iv : bytes/bytearray/memoryview
            The initialization vector to use for encryption or decryption.
            It is as long as the cipher block.

            **The IV must be unpredictable**. Ideally it is picked randomly.

            Reusing the *IV* for encryptions performed with the same key
            compromises confidentiality.

          segment_size : integer
            The number of bytes the plaintext and ciphertext are segmented in.
        """

        self._state = VoidPointer()
        result = raw_cfb_lib.CFB_start_operation(block_cipher.get(),
                                                 c_uint8_ptr(iv),
                                                 c_size_t(len(iv)),
                                                 c_size_t(segment_size),
                                                 self._state.address_of())
        if result:
            raise ValueError("Error %d while instantiating the CFB mode" % result)

        # Ensure that object disposal of this Python object will (eventually)
        # free the memory allocated by the raw library for the cipher mode
        self._state = SmartPointer(self._state.get(),
                                   raw_cfb_lib.CFB_stop_operation)

        # Memory allocated for the underlying block cipher is now owed
        # by the cipher mode
        block_cipher.release()

        self.block_size = len(iv)
        """The block size of the underlying cipher, in bytes."""

        self.iv = _copy_bytes(None, None, iv)
        """The Initialization Vector originally used to create the object.
        The value does not change."""

        self.IV = self.iv
        """Alias for `iv`"""

        self._next = ["encrypt", "decrypt"]

    def encrypt(self, plaintext, output=None):
        """Encrypt data with the key and the parameters set at initialization.

        A cipher object is stateful: once you have encrypted a message
        you cannot encrypt (or decrypt) another message using the same
        object.

        The data to encrypt can be broken up in two or
        more pieces and `encrypt` can be called multiple times.

        That is, the statement:

            >>> c.encrypt(a) + c.encrypt(b)

        is equivalent to:

             >>> c.encrypt(a+b)

        This function does not add any padding to the plaintext.

        :Parameters:
          plaintext : bytes/bytearray/memoryview
            The piece of data to encrypt.
            It can be of any length.
        :Keywords:
          output : bytearray/memoryview
            The location where the ciphertext must be written to.
            If ``None``, the ciphertext is returned.
        :Return:
          If ``output`` is ``None``, the ciphertext is returned as ``bytes``.
          Otherwise, ``None``.
        """

        if "encrypt" not in self._next:
            raise TypeError("encrypt() cannot be called after decrypt()")
        self._next = ["encrypt"]

        if output is None:
            ciphertext = create_string_buffer(len(plaintext))
        else:
            ciphertext = output

            if not is_writeable_buffer(output):
                raise TypeError("output must be a bytearray or a writeable memoryview")

            if len(plaintext) != len(output):
                raise ValueError("output must have the same length as the input"
                                 "  (%d bytes)" % len(plaintext))

        result = raw_cfb_lib.CFB_encrypt(self._state.get(),
                                         c_uint8_ptr(plaintext),
                                         c_uint8_ptr(ciphertext),
                                         c_size_t(len(plaintext)))
        if result:
            raise ValueError("Error %d while encrypting in CFB mode" % result)

        if output is None:
            return get_raw_buffer(ciphertext)
        else:
            return None
```
可以看到他的初始化和`encrypt`方法中调用了`c_uint8_ptr`方法，之所以报错就是因为这个方案抛出了一个异常，他验证了你是不是`bytes`类型，如果你传递的是`bytes`类型他将会走`elif byte_string(data)`分支

```python
    def byte_string(s):
        return isinstance(s, bytes)
    def c_uint8_ptr(data):
        if isinstance(data, _buffer_type):
            # This only works for cffi >= 1.7
            return ffi.cast(uint8_t_type, ffi.from_buffer(data))
        elif byte_string(data) or isinstance(data, _Array):
            return data
        else:
            raise TypeError("Object type %s cannot be passed to C code" % type(data))
```
### 小结
从代码可以明确确认`encrypt`需要传递的是`bytes`类型。其实从`encrypt`函数的注释也可以看出他并不支持`str`类型`plaintext : bytes/bytearray/memoryview`

`pycharm`也有相关的提示![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/31be27d81448320a042ad27a4b1e6619.png)
## Linux
**Linux**的`AES.new`是直接创建一个`AESCipher`类，他们的`encrypt`并不一样
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/8ba9442974eac761222f1ff9fb3c23cf.png)

`AESCipher`是`BlockAlgo`的子类，子类并没有重写`encrypt`方法所以调用的是父类的

```python
class BlockAlgo:
    """Class modelling an abstract block cipher."""

    def __init__(self, factory, key, *args, **kwargs):
        self.mode = _getParameter('mode', 0, args, kwargs, default=MODE_ECB)
        self.block_size = factory.block_size
        
        if self.mode != MODE_OPENPGP:
            self._cipher = factory.new(key, *args, **kwargs)
            self.IV = self._cipher.IV
        else:
            # OPENPGP mode. For details, see 13.9 in RCC4880.
            #
            # A few members are specifically created for this mode:
            #  - _encrypted_iv, set in this constructor
            #  - _done_first_block, set to True after the first encryption
            #  - _done_last_block, set to True after a partial block is processed
            
            self._done_first_block = False
            self._done_last_block = False
            self.IV = _getParameter('iv', 1, args, kwargs)
            if not self.IV:
                raise ValueError("MODE_OPENPGP requires an IV")
            
            # Instantiate a temporary cipher to process the IV
            IV_cipher = factory.new(key, MODE_CFB,
                    b('\x00')*self.block_size,      # IV for CFB
                    segment_size=self.block_size*8)
           
            # The cipher will be used for...
            if len(self.IV) == self.block_size:
                # ... encryption
                self._encrypted_IV = IV_cipher.encrypt(
                    self.IV + self.IV[-2:] +        # Plaintext
                    b('\x00')*(self.block_size-2)   # Padding
                    )[:self.block_size+2]
            elif len(self.IV) == self.block_size+2:
                # ... decryption
                self._encrypted_IV = self.IV
                self.IV = IV_cipher.decrypt(self.IV +   # Ciphertext
                    b('\x00')*(self.block_size-2)       # Padding
                    )[:self.block_size+2]
                if self.IV[-2:] != self.IV[-4:-2]:
                    raise ValueError("Failed integrity check for OPENPGP IV")
                self.IV = self.IV[:-2]
            else:
                raise ValueError("Length of IV must be %d or %d bytes for MODE_OPENPGP"
                    % (self.block_size, self.block_size+2))

            # Instantiate the cipher for the real PGP data
            self._cipher = factory.new(key, MODE_CFB,
                self._encrypted_IV[-self.block_size:],
                segment_size=self.block_size*8)

    def encrypt(self, plaintext):
        """Encrypt data with the key and the parameters set at initialization.
        
        The cipher object is stateful; encryption of a long block
        of data can be broken up in two or more calls to `encrypt()`.
        That is, the statement:
            
            >>> c.encrypt(a) + c.encrypt(b)

        is always equivalent to:

             >>> c.encrypt(a+b)

        That also means that you cannot reuse an object for encrypting
        or decrypting other data with the same key.

        This function does not perform any padding.
       
         - For `MODE_ECB`, `MODE_CBC`, and `MODE_OFB`, *plaintext* length
           (in bytes) must be a multiple of *block_size*.

         - For `MODE_CFB`, *plaintext* length (in bytes) must be a multiple
           of *segment_size*/8.

         - For `MODE_CTR`, *plaintext* can be of any length.

         - For `MODE_OPENPGP`, *plaintext* must be a multiple of *block_size*,
           unless it is the last chunk of the message.

        :Parameters:
          plaintext : byte string
            The piece of data to encrypt.
        :Return:
            the encrypted data, as a byte string. It is as long as
            *plaintext* with one exception: when encrypting the first message
            chunk with `MODE_OPENPGP`, the encypted IV is prepended to the
            returned ciphertext.
        """

        if self.mode == MODE_OPENPGP:
            padding_length = (self.block_size - len(plaintext) % self.block_size) % self.block_size
            if padding_length>0:
                # CFB mode requires ciphertext to have length multiple of block size,
                # but PGP mode allows the last block to be shorter
                if self._done_last_block:
                    raise ValueError("Only the last chunk is allowed to have length not multiple of %d bytes",
                        self.block_size)
                self._done_last_block = True
                padded = plaintext + b('\x00')*padding_length
                res = self._cipher.encrypt(padded)[:len(plaintext)]
            else:
                res = self._cipher.encrypt(plaintext)
            if not self._done_first_block:
                res = self._encrypted_IV + res
                self._done_first_block = True
            return res

        return self._cipher.encrypt(plaintext)
```
这里`encrypt`返回的是`self._cipher.encrypt(plaintext)`,这个`self._cipher`是[内建对象](https://docs.python.org/zh-cn/3/library/builtins.html)，他的加密实际上是**c语言**写的，**c语言**并没有`byte string`这个概念，在**C语言**中，字符串本质上就是一连串的字节（**char**类型的数组），以`'\0'`（空字符）作为字符串的终止符。因此，一个常规的字符串在**C语言**中本质上已经是一个字节字符串。这应该就是**Linux**版本的加密输入字符串不会报错的根本原因。(猜测**Windows** 使用 `python2` 也不会报错,因为在 `Python 2` 中字符串和字节是同一回事)

### 小结
虽然**Linux**版本的**sdk**输入字符串不会报错，但是他的**python**代码注释明确指出输入是`byte string`，所以虽然`c语言`的处理使得程序不会报错，但是我觉得可以直接传入一个`bytes`类型不会与文档注释产生歧义

# 总结
个人觉得在编写代码时可以输入一个`bytes`类型，这样可以省去跨平台问题，而且在`AES.new`的时候传参都是`bytes`，最后`encrypt`时的**plaintext**却是`str`类型不会感觉很怪吗

```python
from Crypto.Cipher import AES
AES.new(b'zXP5GtJMmtpw2pIiwCpFcg7L', AES.MODE_CFB, b'zXP5GtJMmtpw2pIi').encrypt("plaintext".encode())
```
或者在代码中加入Windows平台兼容，避免也许是我上面的推导错误传入`bytes`类型会产生某种未知错误导致代码报错
```python
from Crypto.Cipher import AES
plaintext = "plaintext"
import sys
if sys.platform == "win32":
    # windows 下使用 AES 加密需要转为bytes类型(win使用 CfbMode类, Linux 使用 AESCipher类)
    plaintext = plaintext.encode()
AES.new(b'zXP5GtJMmtpw2pIiwCpFcg7L', AES.MODE_CFB, b'zXP5GtJMmtpw2pIi').encrypt(plaintext.encode())
```

# 划重点
这个原始库都不维护了，他官方都觉得有一堆bug推荐别的了
![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/ca0e85aee25d17600c0cef7361443b7e.png)
## 参考链接
[https://www.pycrypto.org/](https://www.pycrypto.org/)
[github](https://github.com/pycrypto/pycrypto/issues/238)
