### Prompt: given an integer find the sum of each of the digits in the int

Using base-10, any integer `n` can be written as:

```
n = 10*q + r
```

where:

- `q` is the integer quotient when dividing by 10 (drop the fractional part),
    
- `r` is the remainder, and `0 ≤ r ≤ 9`.
    

In C:

- `r` is `n % 10` (the last digit)
    
- `q` is `n / 10` (integer division → removes the last digit)
    

So in a loop:

1. grab the last digit: `digit = n % 10`
    
2. add/use it
    
3. chop it off: `n = n / 10`
    

Repeat until `n` becomes 0.

### Tiny trace: `n = 12345`

- `n % 10 = 5`, `n / 10 = 1234`
    
- `n % 10 = 4`, `n / 10 = 123`
    
- `n % 10 = 3`, `n / 10 = 12`
    
- `n % 10 = 2`, `n / 10 = 1`
    
- `n % 10 = 1`, `n / 10 = 0` → stop
    

### Code sketch

```c
long long sum_digits(long long n) {
    n = llabs(n);          // handle negatives; % and / with negatives are defined but this is simpler
    long long sum = 0;
    while (n > 0) {
        sum += n % 10;     // peel last digit
        n /= 10;           // drop last digit
    }
    return sum;            // (returns 0 for input 0)
}
```

### Generalization (any base `b`)

- last “digit” in base `b`: `n % b`
    
- remove last digit: `n /= b`
    

That’s the whole “peel off digits” trick.