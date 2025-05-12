# Writeup RSA-customer-service

Challenge author : nikost | Writeup author : dolipr4necrypt0

## Challenge analysis

At first glance, this challenge looks like a standard RSA setup with `N=p*q`, `phi=(p-1)*(q-1)` and `d=e^-1 mod phi`. But there's a twist: for this structure to work, `p` and `q` must be prime which is not the case. That means the Euler totient `phi` is incorrect and the decryption fails.

Initially, I had to think quite a bit about the challenge. But once the trick became clear, I understood why it was originally categorized as "Easy".

My first thought was: if we can fully factor N, we can compute the correct `phi`, then invert `e` and `d`, allowing us to recover the original message. However, full factorization isn't feasible here. We can identify some small factors, but a large composite factor remains, as shown in the FactorDB factorization:
`4892129385...10<614> = 2 · 5 · 163 · 8263 · 11617 · 11909899 · 1515847349<10> · 12600377473<11> · 37984123211381<14> · 7775904744864652956285367<25> · 4653486384...77<538>`

The last factor is a 538-digit composite number, but too large to factor easily using basic tools.

## The breakthrough

I was stuck for a while. Without factoring `N`, we can't compute `phi`, which seemed like a dead end. But then a hint was released, suggesting that partial factorization might be enough to solve the challenge.

I needed a little time again to understand why it was the case. And it works because the message is small, the flag as a number is smaller than the product of the primes that we found in the facotrisation of `N`. We can reduce everything mod the product of the small factors, the message stay the same because smaller than the product and we can use the original strategy.

Let's define `M=p1*...*p9` where `p1`, ... , `p9` are the primes in the factorisation of `N`. Now we can define `phi=(p1-1)*...*(p9-1)`. I started to implement the strategy, I invert `d` modulo `phi`. But I have an error `d` is not invertible modulo `phi`. Back to the start.

## Using the Chinese Reminder Theorem

What we need is to find solutions to `x^d=c mod M`, which have a unique solution when `gcd(d,phi)=1` but in general we can have multiples solutions. By the Chinese Reminder Theorem, we can find those solutions by solving a system of equations for each prime :
```
x^d=c mod p1
....
x^d=c mod p9
```

For small values of `p`, we can bruteforce to find solutions and we find indeed severals solutions for the primes for which `gcd(d,p-1)!=1`.

For large primes, bruteforcing isn't practical, but we can use SAGE `nth_root` function to find solutions. Once we collect all the roots modulo each prime, we combine them using the CRT to reconstruct full solutions modulo `M`.

Then we just check each candidate to see if it contains the flag.

## Final script

The only thing that we added in the final script is that we compute the solutions of `x^(e*d)=c mod p` instead of just `x^d mod p`.

```python
factors=[2,5,163,8263,11617,11909899,1515847349,12600377473,37984123211381,7775904744864652956285367]
sols=[]
for f in factors:
    sols.append(Zmod(f)(m).nth_root(e*d, all=True))

for p in list(product(*sols)):
    x=CRT([int(x) for x in list(p)], factors)
    if b"RM" in long_to_bytes(x):
        print(long_to_bytes(x))
```

And we have our flag : RM{prime_suspect_was_composite}.
