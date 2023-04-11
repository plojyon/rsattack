[//]: # (Predogled na https://github.com/plojyon/rsattack)

# a

Zasifrirajmo podatke po dani formuli z Bojanovimi podatki.

```python
>>> u = 34100010166843172
>>> i = 1909
>>> m = (u << 123) + i
>>> n = 1353040922319896710729948440742113526140662069124237571
>>> e = 3
>>> pow(m, e, n)  # m^e % n
723941917581347359534709894322925645398915405217429002
```

Bojan poslje na streznik stevilo `723941917581347359534709894322925645398915405217429002`

# b
```python
>>> c = [None for _ in range(3)]
>>> c[0] = 164867525413631686108542244605590332657131844994186648
>>> c[1] = 669330273667331356154438891368274712878935740757554990
>>> c[2] = 853109242122207805055956523445529343243869885591747755
```
Definirajmo `g := (u*2^123 + j)`. Z binomskim izrekom izracunamo naslednje identitete (izpuscam mod n, ker je povsod):
```latex
c_j = g^3
c_{j+1} = (u*2^123 + j + 1)^3
        = (1 + 3g + 3g^2 + g^3)
c_{j+2} = (u*2^123 + j + 2)^3
        = (8 + 12g + 6g^2 + g^3)

x = (c_{j+1} - c_{j} - 1)/3 = g + g^2
y = (c_{j+2} - c_{j+1} - 7)/3 = 3g + g^2

g = ((y - x) / 2)
g = (c_{j+2} - 2c_{j+1} + c_{j} - 6)/6
```

in jih izracunajmo:

```python
>>> g = ((c[2] - 2*c[1] + c[0] - 6) % n) // 6
>>> g
172059523753512248264261571009447296047298719699176998
>>> g**3 % n == c[0]
True
```

Na koncu smo preverili, da je `c_0 == g^3`, torej smo dobili pravilno.

Spomnimo se na definicijo `g := (u*2^123 + j)`, torej `u = g >> 123`
```python
>>> u = g >> 123
>>> u
16180399854194147
```

Anina skrivnost je torej `16180399854194147`.

# c

Ker ima skrivnost samo 30 bitov, lahko izvedemo napad z grobo silo (vseh moznosti je `2^30 ~ (2^10)^3 ~ 10^9`, kar je praviloma izvedljivo).

```python
>>> from tqdm import tqdm  # dynamic progress bar
>>> n = 1353040922319896710729948440742113526140662069124237571
>>> e = 3
>>> s = 661795599945365472793374
>>> c = 1304427715737794183639527703843118636234043562220564999
>>> for u in tqdm(range(s<<99, (s<<99) + (1<<31))):
...   if ((u*u % n) * u) % n == c: print("\nFound: " + str(u))
 47%|████▋     | 1003566057/2147483648 [21:48<24:46, 769515.48it/s]
Found: 419462794749571861247243308751354772800620294170442904
100%|██████████| 2147483648/2147483648 [47:28<00:00, 753955.95it/s]
```

Za hitrejse delovanje sem iteriral od `(s << 99)` do `((s << 99) + (1 << 31))`, da se izognem dodatnemu sestevanju v zanki.
Z `((u*u % n) * u) % n` izracunam `u^3 mod n`, ker je to hitreje kot `pow(u, 3, n)`.

Kljub temu sem bil nekoliko razocaran nad pocasnostjo, saj sem na rezultat cakal skoraj 22 minut, za vse iteracije pa kar 47 min.
Process je seveda paralelizabilen, cesar zaradi enostavnosti nisem izkoriscal.

```python
>>> m = 419462794749571861247243308751354772800620294170442904
>>> bin(m)
'0b10001100001001000000001011101000011111101111001001000000010100011101001100011110000000000000000000000000000000000000000000000000000000000000000000000111011110011110111110010011000'
```

V binarnem zapisu lepo vidimo locitev med `s` in `u`.

```python
>>> u = m % (1 << 30)
>>> u
1003453592
```

Danino skrivno stevilo je `1003453592`.
