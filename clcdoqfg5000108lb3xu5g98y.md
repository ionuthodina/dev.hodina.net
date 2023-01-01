# Given 2 big numbers, what is their modulus?

### Intro

I received this puzzle in one of my interview coding assignments, and I particularly enjoyed it. Even if the statement sounded quite hard at the beginning, it turned out to have a simple approach. Below is my rough solution.

### The puzzle

Indeed, I don't remember exactly the statement as I forgot to save it, but lucky enough, I have the code and can reverse engineer it! 😎

> Given a number N, and a list of integers a\[0\],.., a\[N-1\], calculate the index *i* of the highest modulus between a\[i\] raised power to a\[i+1\] and 1 000 000 007. The formula is as follows:

```csharp
result[i] = (a[i]^a[i+1]) % (10^9 + 7)
```

### The approach

The first member of the modulus can get extremely big. The second one is not so small either. Even if a programming language would support such an operation with big numbers, the idea is to come up with a clever solution that does not need special context for solving it. One way to solve this is to deconstruct the Base and Power recursively until numbers are small enough to calculate modulus.

```csharp
// deconstruct Power
(Base^Power % N) = ((Base^(Power/2) % N) * (Base^(Power - Power/2) % N)) %N

//deconstruct Base, using p as a prime number
(Base^Power % N) = ((p^Power % N) * ((Base/p)^Power % N)) %N
```

### Full source code

```csharp
public static int GetIndexOfHighestModulus(List<int> a)
{
	long highestNumber = 0;
	int index = 0;

	for (int i = 0; i < a.Count - 1; i++)
	{
		long modulo = GetMod(a[i], a[i + 1]);
		if (modulo > highestNumber)
		{
			highestNumber = modulo;
			index = i;
		}
	}

	return index+1;
}

private static long GetMod(int Base, int Power)
{
	if (Power > 8)
	{
		var result = GetMod(GetMod(Base, Power / 2) * GetMod(Base, Power - Power / 2));
		return result;
	}

	var primeNumbers = new List<int> { 2, 3, 5, 7, 11, 13, 17, 19, 23, 29, 31, 37, 41, 43, 47, 53, 59, 61, 67, 71, 73, 79, 83, 89, 97 };
	if (!primeNumbers.Contains(Base))
	{
		foreach (var primeNumber in primeNumbers)
		{
			if (Base % primeNumber == 0)
			{
				var result = GetMod(GetMod(primeNumber, Power) * GetMod(Base / primeNumber, Power));
				return result;
			}
		}
	}

	var modulo = GetMod((long)Math.Pow(Base, Power));
	return modulo;
}

private static long GetMod(long number)
{
	return number % ((long)Math.Pow(10, 9) + 7);
}
```

### Voilà

With just a few basic formulas we have an elegant solution 🤩. Don't just copy-paste this and let your future employer where you got inspiration from, 😏!

Did you find a better solution? Please let me know💡.

Happy coding! 💻