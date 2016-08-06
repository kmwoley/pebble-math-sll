# pebble-math-sll
A fixed point (31.32 bit) math library for Pebble smart watches.

# Purpose
Pebble is natively bad at emulating handling floating point - using standard C `double` and `float` types bloat Pebble code and perform very slowly. This library provides an alternative to using native C floating point.

Floating point packs the most accuracy in the available bits, but it often provides more accuracy than is required. It is time consuming to carry the extra precision around, particularly on platforms that don't have a dedicated floating point processor like the Pebble.

The math-sll library is a compromise.  All math is done using the 64 bit signed "long long" format (sll), and is not intended to be portable, just as simple and as fast as possible.

Since "long long" is a elementary type, it can be passed around without resorting to the use of pointers.  Since the format used is fixed point, there is never a need to do time consuming checks and adjustments to maintain normalized numbers, as is the case in floating point.
 
# Limitations
1. Range. This library is limited to handling numbers with a whole part of up to 2^31 - 1 = 2.147483647e9 in magnitude, and fractional parts down to 2^-32 = 2.3283064365e-10 in magnitude. This yields a decent range and accuracy for many applications.
2. No checking for arguments out of range (error).
3. No checking for divide by zero (error).
4. No checking for overflow (error).
5. No checking for underflow (warning).
6. Chops, doesn't round.

# Getting Started

1. Run `pebble package install pebble-math-sll` from your project's directory to install the package in your project
2. Add `ctx.env.CFLAGS.append('-Wa,-mimplicit-it=thumb')` to your `wscript` file to enable proper ARM assembly compilation. Example:
    ```
    ...
    for p in ctx.env.TARGET_PLATFORMS:
        ctx.set_env(ctx.all_envs[p])
        ctx.set_group(ctx.env.PLATFORM_NAME)
        
        # pebble-math-sll arm assembly requirements
        ctx.env.CFLAGS.append('-Wa,-mimplicit-it=thumb')

        app_elf = '{}/pebble-app.elf'.format(ctx.env.BUILD_DIR)
        ctx.pbl_program(source=ctx.path.ant_glob('src/**/*.c'), target=app_elf)
        ...
    ```
3. Include `<pebble-math-sll/math-sll.h>` as needed.
 
# Functions
*For details on functions and usage, see [`math-sll.h`](https://github.com/aemileski/math-sll/blob/master/math-sll.h)*

# Examples
Here is an example of absolute value (abs) utilizing math-sll.
```
sll sllabs(sll x) {
  sll ret = x;
  if (x < CONST_0) {
    ret = sllneg(x);
  }
  return ret;
}
```

Here is an example of an atan2 implementation utilizing math-sll.
```
/** http://math.stackexchange.com/questions/1098487/atan2-faster-approximation
* atan2(y,x)
* a := min (|x|, |y|) / max (|x|, |y|)
* s := a * a
* r := ((-0.0464964749 * s + 0.15931422) * s - 0.327622764) * s * a + a
* if |y| > |x| then r := 1.57079637 - r
* if x < 0 then r := 3.14159274 - r
* if y < 0 then r := -r
**/
sll sllatan2(sll y, sll x) {
  sll abs_x = sllabs(x);
  sll abs_y = sllabs(y);
  sll maxyx = max(abs_x, abs_y);
  sll minyx = min(abs_x, abs_y);

  sll a = slldiv(minyx, maxyx);
  sll s = sllmul(a, a);
  sll r_1 = sllmul(dbl2sll((double)(-0.0464964749)), s);
  sll r_2 = slladd(r_1, dbl2sll((double)0.15931422));
  sll r_3 = sllsub(sllmul(r_2, s), dbl2sll((double)0.327622764));
  sll r = slladd(sllmul(sllmul(r_3, s), a), a);

  if(sllabs(y) > sllabs(x)) {
    r = sllsub(dbl2sll((double)1.57079637), r);
  }
  if(x < CONST_0) {
    r = sllsub(dbl2sll((double)3.14159274), r);
  }
  if(y < CONST_0) {
    r = sllneg(r);
  }

  return r;
}
```

# Source & Credits
This is an implementation of Andrew E. Mileski <<andrewm@isoar.ca>>'s [math-sll](https://github.com/aemileski/math-sll) library tailored to Pebble smart watches. See source for additional credits.

The source for this [Pebble package](https://developer.pebble.com/guides/pebble-packages/) can be found on GitHub at [pebble-math-sll](https://github.com/kmwoley/pebble-math-sll/) and is maintained by Kevin Michael Woley <<kmwoley@gmail.com>>. Contributions welcome.

# License
 Licensed under the terms of the MIT license:
 
 Original work Copyright (c) 2000,2002,2006,2012,2016 Andrew E. Mileski <<andrewm@isoar.ca>>
 
 Modified work Copyright (c) 2016 Kevin Michael Woley <<kmwoley@gmail.com>>
 
 Permission is hereby granted, free of charge, to any person obtaining a copy of this software and associated documentation files (the "Software"), to deal in the Software without restriction, including without limitation the rights to use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies of the Software, and to permit persons to whom the Software is furnished to do so, subject to the following conditions:

The copyright notice, and this permission notice shall be included in all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.
