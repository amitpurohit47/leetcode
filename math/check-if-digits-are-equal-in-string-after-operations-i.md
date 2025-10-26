# Problem Link
https://leetcode.com/problems/check-if-digits-are-equal-in-string-after-operations-i

# Intuition
Initially I was clueless about how to solve this. Then went through multiple explanations and got to know the only way to solve this in $O(n)$ is to use **Lucas' Theorem** and **Chinese Remainder Theorem** . 

# Approach
<!-- Describe your approach to solving the problem. -->
The first question that comes to mind is how to build a thought process to this problem. If we solve a few examples by hand and simulate the process we get to know we need to just multiply each digit is contributing a finite number of times to the final result. Now how to find the contribution by each digit. 

$\quad1 \quad 4\quad 6\quad 4\quad 1\\\\\quad\quad1\quad3\quad 3\quad 1 \\\\ \quad\quad\quad1\quad 2\quad1 \\\\ \quad\quad\quad\quad1\quad1 \\\\ \quad\quad\quad\quad\quad1$

Here there a final result which contibutes 1 times to the solution. It came from 1 contribution from each of 2 digits from earlier operation. If we go to the next layer, the leftmost and rightmost have just 1 contribution to the layer below and the middle digit contibutes once to each digit, so 2 contributions from the middle digit. If we check this until we reach the size of original problem  we can observe that each digit has a coffecient that corresponds to a coefficient in binomial expansion of $(1 + x)^{n - 1}$.

Just to clarify, we can prove that 
- Only 2 digits remain in the end
- The first digit of the end result doesn't get any contribution from last digit of original array 
- The second digit of the end result doesn't get any contribution from the first digit of the original array

Concluding the initial though process
$firstDigit = \sum_{i = 0, r = 0}^{n - 2, n - 2}` \binom{n-2}{r} * s[i]\ \%\ 10$ and 
$secondDigit = \sum_{i = 1, r = 0}^{n - 1, n - 2} \binom{n-2}{r} * s[i]\ \%\ 10$

Now the main challenge is to calculate $\binom{n - 2}{r} \ \% \ 10$ efficiently because the result of the combination can be horrifyingly huge for $len(s) = 10^5$.

We can try to break this problem into easier problems. 
We have $10 = 2 * 5$. Both  $2$ and $5$ are prime, but $10$ is not.

**Lucas' Theorem** states that,  
To find large 
$\binom{n}{r}\ \% \ p$, if $p$ is a prime number, then we can break the large combination into parts and calculate the final result.

$`\binom{n}{r} \ \% \ p = ( \ \binom{n_0}{r_0} \ * \ \binom{n_1}{r_1} \ * \ \binom{n_1}{r_2} \ ... \ \binom{n_k}{r_k}  \ ) \ \% \ p`$

Where $n_0,n_1,...n_k$ are coefficients of the $base \ p$ equivalent of $n$ and Where $r_0,r_1...r_k$ are coefficients of the $base \ p$ equivalent of $r$.

We calculate $\binom{n}{r} \ \% \ 2$ and $\binom{n}{r} \ \% \ 5$ and store their results.


**Chinese Remainder Theorem** states that,

Given,
$\\ x ≡  a_0 \ (mod \ n_0) \\
x ≡  a_1 \ (mod \ n_1) \\
\\
x ≡  a_k \ (mod \ n_k)$ 

Where $a_0, a_1,...,a_k$ are pairwise coprime, then there exists a unique solution and the solution is 

$x \ = \ \sum a_i,N_i,z_i \ \% \ N$

Where $N=  n_0 . n_1 ... n_k , \ N_i = \frac{N}{n_0 . n_1 ... n_{i - 1} . n_{i + 1} ... n_k}$ and $z_i = N^{-1}_i$

In context of this problem, we have 

$x ≡ a_0 \ (mod \ 2) \\
\,x ≡ a_1 \ (mod \ 5)$

$2$ and $5$ are pair wise coprime and we need the final result $mod \ 10$, so this perfectly fits our needs. We get the equation $x = (5a_0 \ + \ 6a_1) \ \% \ 10$ for our problem by solving for $x$ using CRT.

Please refer to code for the complete algorithm broken down into smaller and steps.


<!-- then we can write $$ n $$ and $$ r $$, in $$ base\  p  $$ equivalent -->



# Code
```cpp []
class Solution {
public:

    map<pair<int, int>, int> comb;

    // This is store smaller predetermined combination results that will be required during Lucas' theorem execution
    // Since Base 5 equivalent's coefficients will be at max 4, we can store all possible coefficients before hand
    // For base 2 equivalents only 4 possible combinations are possible
    Solution() {
        comb[{0,0}] = 1;
        comb[{0,1}] = 0;
        comb[{0,2}] = 0;
        comb[{0,3}] = 0;
        comb[{0,4}] = 0;
        comb[{1,0}] = 1;
        comb[{2,0}] = 1;
        comb[{3,0}] = 1;
        comb[{4,0}] = 1;
        comb[{1,1}] = 1;
        comb[{2,1}] = 2;
        comb[{3,1}] = 3;
        comb[{4,1}] = 4;
        comb[{2,2}] = 1;
        comb[{3,2}] = 3;
        comb[{4,2}] = 6;
        comb[{3,3}] = 1;
        comb[{4,3}] = 4;
        comb[{4,4}] = 1;
    }

    // Convert number to base 2 equivalent
    string getBase2(int n) {
        string s = "";
        while ( n ) {
            s += (n % 2 + '0');
            n /= 2;
        }
        reverse(s.begin(), s.end());
        return s;
    } 

    // Convert number to base 5 equivalent
    string getBase5(int n) {
        string s = "";
        while ( n ) {
            s += ( n % 5 + '0');
            n /= 5;
        }
        reverse(s.begin(), s.end());
        return s;
    }

    // Applying Lucas's theorem for base 2
    int getResMod2(int n, int r) {
        string nBase2 = getBase2(n);
        string rBase2 = getBase2(r);
        int i = nBase2.length() - 1, j = rBase2.length() - 1;
        int res = 1;
        while ( i >= 0 && j >= 0 ) {
            res = (res * comb[{nBase2[i] - '0', rBase2[j] - '0'}]) % 10;
            i--;
            j--;
        }

        while ( i >= 0 ) {
            res = (res * comb[{nBase2[i] - '0', 0}]) % 10;
            i--;
        }

        return res;
    }

    // Applying Lucas's theorem for base 5
    int getResMod5(int n, int r) {
        string nBase5 = getBase5(n);
        string rBase5 = getBase5(r);
        int i = nBase5.length() - 1, j = rBase5.length() - 1;
        int res = 1;
        while ( i >= 0 && j >= 0 ) {
            res = (res * comb[{nBase5[i] - '0', rBase5[j] - '0'}]) % 10;
            i--;
            j--;
        }

        while ( i >= 0 ) {
            res = (res * comb[{nBase5[i] - '0', 0}]) % 10;
            i--;
        }

        return res;
    }

    // Applying Chinese Remainder Theorem to merge the results
    int getResMod10(int resMod2, int resMod5) {
        return (5 * resMod2 + 6 * resMod5) % 10;
    }

    int nCr(int n, int r) {
        int resMod2 = getResMod2(n, r);
        int resMod5 = getResMod5(n, r);
        return getResMod10(resMod2, resMod5);
    }

    int getSumAfterOperations(string &s, int ind) {
        int r = 0, sum = 0, n = s.length();
        for ( int i = ind; r < n - 1; i++  ) {
            sum = (sum + ((s[i] - '0') * nCr(n - 2, r)) % 10) % 10;
            r++;
        }
        return sum;
    }

    bool hasSameDigits(string s) {
        int firstDigit = getSumAfterOperations(s, 0);
        int secondDigit = getSumAfterOperations(s, 1);
        return firstDigit == secondDigit;
    }
};
```

# Complexity
- Time complexity: $O(n*log(len(s)))$
We are iterating over the string twice(one for each digit in final result). For each interation only the work done for Lucas' Theorem execution is considered.
<!-- Add your time complexity here, e.g. $$O(n)$$ -->

- Space complexity: $O(log(len(s)))$
The only extra space required is for calculating base 2 and base 5 equivalents

# References
- https://brilliant.org/wiki/chinese-remainder-theorem/
- https://brilliant.org/wiki/lucas-theorem/
<!-- Add your space complexity here, e.g. $$O(n)$$ -->
