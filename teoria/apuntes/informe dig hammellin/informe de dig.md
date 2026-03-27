# informe del uso de dig y captura de paquetes

use dig para hacer consultas dns y catchear en wireshark que te responde, 

## consulta recursiva

```sh
dig bocajuniors.com.ar +trace
```

![consulta recursiva en dig](consulta%20recursiva.png)

![consulta recursiva en wireshark](consulta%20recursiva%20en%20wireshark.png)

## consulta autoritativa

```sh
dig @ns1.huaweicloud-dns.net bocajuniors.com.ar +norecurse
```
![consulta autoritativa en dig](consulta%20autoritativa.png)

![consulta autoritativa en wireshark](consulta%20autoritativa%20wireshark.png)


## consulta verbose

```sh
dig @ns1.huaweicloud-dns.net bocajuniors.com.ar +norecurse +qr +all +multiline
```

![consulta verbose en dig](consulta%20verbose.png)