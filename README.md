# fp

Functions for parsing and converting floating points to/from strings in their shortest, most accurate representation.

## Functions

```c
/// Convert a double to a string
size_t fp_dtoa(double d, char fmt, char *dst, size_t nbytes);
/// Convert a float to a string
size_t fp_ftoa(float d, char fmt, char *dst, size_t nbytes);
/// Convert a string to a double
bool fp_atod(char *str, size_t len, double *x);
/// Convert a string to a float
bool fp_atof(char *str, size_t len, float *x);
/// Parse floating point info from a data stream and return useful details.
struct fp_info fp_parse(char *str, size_t len);
```

These functions are often faster and more accurate than the ones that are 
provided by your standard C compiler, eg. `atof`, `atod`, and `printf`.
This is mostly due to the excellent conversion code provided by the
[ulfjack/ryu](https://github.com/ulfjack/ryu) project.

Also included is a fast number parser `fp_parse` for reading an IEEE+754
details from a stream of bytes. This is useful when needing to validate
floating points from formats such as JSON, WKT, and SQL.

## Quality

Here's a comparison of the quality of the conversions using the `fp` library
to the standard C printf.

For the following floating points:

```
1230456078.90123e10
-7750609.598126
19688.27920015702
731575400000
97967.0432226889
1216277862.336918
```

Using 'g' format:

```
+---------------------+--------------+
|         fp          |    printf    |
+---------------------+--------------+
| 1.23045607890123e19 | 1.23046e+19  |
| -7750609.598126     | -7.75061e+06 |
| 19688.27920015702   | 19688.3      |
| 7.315754e11         | 7.31575e+11  |
| 97967.0432226889    | 97967        |
| 1216277862.336918   | 1.21628e+09  |
+---------------------+--------------+
```

Using 'e' format:

```
+---------------------+---------------+
|         fp          |     printf    |
+---------------------+---------------+
| 1.23045607890123e19 | 1.230456e+19  |
| -7.750609598126e6   | -7.750610e+06 |
| 1.968827920015702e4 | 1.968828e+04  |
| 7.315754e11         | 7.315754e+11  |
| 9.79670432226889e4  | 9.796704e+04  |
| 1.216277862336918e9 | 1.216278e+09  |
+---------------------+---------------+
```

Using 'f' format:

```
+----------------------+-----------------------------+
|         fp           |            printf           |
+----------------------+-----------------------------+
| 12304560789012300000 | 12304560789012299776.000000 |
| -7750609.598126      | -7750609.598126             |
| 19688.27920015702    | 19688.279200                |
| 731575400000         | 731575400000.000000         |
| 97967.0432226889     | 97967.043223                |
| 1216277862.336918    | 1216277862.336918           |
+----------------------+-----------------------------+
```

## Using

Just drop the `fp.c` and `fp.h` files into your project.

## Examples

Convert a double to a string.

```c
char buf[32];
size_t n = fp_dtoa(-112.89123883, 'f', buf, sizeof(buf));
if (n >= sizeof(buf)) {
	// Buffer is too small to store the entire floating point as a string.
    abort();
}
printf("%s\n", buf);

// Output: -112.89123883
```

Parse a double precision floating point, and then convert it back

```c
char *str = "-112.89123883";
double x;
bool ok = fp_atod(str, strlen(str), &x);
if (!ok) {
    // Invalid number
    abort();
}
char buf[32];
fp_dtoa(x, 'f', buf, sizeof(buf));
printf("%s\n", buf);

// Output: -112.89123883
```

Looping over a list of line-delimited floating point string using the 
`fp_parse()` function.

```c
char *list = 
    "1230456078.90123e10\n"
    "-7750609.598126\n"
    "19688.27920015702\n"
    "731575400000\n"
    "97967.0432226889\n"
    "1216277862.336918\n";

size_t list_len = strlen(list);
for (size_t i = 0; i < list_len; i++) {
    struct fp_info info = fp_parse(list+i, list_len-i);
    if (!info.ok) {
        // Number is invalid
        abort();
    }
    printf("%.*s\n", (int)info.len, list+i);
    i += info.len;
}

// 1230456078.90123e10
// -7750609.598126
// 19688.27920015702
// 731575400000
// 97967.0432226889
// 1216277862.336918
```

The `struct fp_info` type store the additional information about the floating
point string.

```c
struct fp_info {
    bool ok;     // number is valid
    bool sign;   // has sign. Is a negative number
    size_t frac; // has dot. Index of '.' or zero if none
    size_t exp;  // has exponent. Index of 'e' or zero if none
    size_t len;  // number of bytes parsed
};
```


