---
title: BigInteger类
tags:
  - Java
originContent: ''
categories:
  - 文章
toc: false
date: 2019-09-21 19:00:03
---

# BigInteger类

![image.png](/images/2019/09/21/f1ae7690-dc5e-11e9-8024-c70d44c51336.png)

```java
//
// Source code recreated from a .class file by IntelliJ IDEA
// (powered by Fernflower decompiler)
//

package java.math;

import java.io.IOException;
import java.io.ObjectInputStream;
import java.io.ObjectOutputStream;
import java.io.ObjectStreamField;
import java.io.StreamCorruptedException;
import java.io.ObjectInputStream.GetField;
import java.io.ObjectOutputStream.PutField;
import java.util.Arrays;
import java.util.Objects;
import java.util.Random;
import java.util.concurrent.ThreadLocalRandom;
import jdk.internal.HotSpotIntrinsicCandidate;
import jdk.internal.misc.Unsafe;

public class BigInteger extends Number implements Comparable<BigInteger> {
    final int signum;
    final int[] mag;
    private int bitCountPlusOne;
    private int bitLengthPlusOne;
    private int lowestSetBitPlusTwo;
    private int firstNonzeroIntNumPlusTwo;
    static final long LONG_MASK = 4294967295L;
    private static final int MAX_MAG_LENGTH = 67108864;
    private static final int PRIME_SEARCH_BIT_LENGTH_LIMIT = 500000000;
    private static final int KARATSUBA_THRESHOLD = 80;
    private static final int TOOM_COOK_THRESHOLD = 240;
    private static final int KARATSUBA_SQUARE_THRESHOLD = 128;
    private static final int TOOM_COOK_SQUARE_THRESHOLD = 216;
    static final int BURNIKEL_ZIEGLER_THRESHOLD = 80;
    static final int BURNIKEL_ZIEGLER_OFFSET = 40;
    private static final int SCHOENHAGE_BASE_CONVERSION_THRESHOLD = 20;
    private static final int MULTIPLY_SQUARE_THRESHOLD = 20;
    private static final int MONTGOMERY_INTRINSIC_THRESHOLD = 512;
    private static long[] bitsPerDigit = new long[]{0L, 0L, 1024L, 1624L, 2048L, 2378L, 2648L, 2875L, 3072L, 3247L, 3402L, 3543L, 3672L, 3790L, 3899L, 4001L, 4096L, 4186L, 4271L, 4350L, 4426L, 4498L, 4567L, 4633L, 4696L, 4756L, 4814L, 4870L, 4923L, 4975L, 5025L, 5074L, 5120L, 5166L, 5210L, 5253L, 5295L};
    private static final int SMALL_PRIME_THRESHOLD = 95;
    private static final int DEFAULT_PRIME_CERTAINTY = 100;
    private static final BigInteger SMALL_PRIME_PRODUCT = valueOf(152125131763605L);
    private static final int MAX_CONSTANT = 16;
    private static BigInteger[] posConst = new BigInteger[17];
    private static BigInteger[] negConst = new BigInteger[17];
    private static volatile BigInteger[][] powerCache;
    private static final double[] logCache;
    private static final double LOG_TWO = Math.log(2.0D);
    public static final BigInteger ZERO;
    public static final BigInteger ONE;
    public static final BigInteger TWO;
    private static final BigInteger NEGATIVE_ONE;
    public static final BigInteger TEN;
    static int[] bnExpModThreshTable;
    private static String[] zeros;
    private static int[] digitsPerLong;
    private static BigInteger[] longRadix;
    private static int[] digitsPerInt;
    private static int[] intRadix;
    private static final long serialVersionUID = -8287574255936472291L;
    private static final ObjectStreamField[] serialPersistentFields;

    public BigInteger(byte[] val, int off, int len) {
        if (val.length == 0) {
            throw new NumberFormatException("Zero length BigInteger");
        } else {
            Objects.checkFromIndexSize(off, len, val.length);
            if (val[off] < 0) {
                this.mag = makePositive(val, off, len);
                this.signum = -1;
            } else {
                this.mag = stripLeadingZeroBytes(val, off, len);
                this.signum = this.mag.length == 0 ? 0 : 1;
            }

            if (this.mag.length >= 67108864) {
                this.checkRange();
            }

        }
    }

    public BigInteger(byte[] val) {
        this((byte[])val, 0, val.length);
    }

    private BigInteger(int[] val) {
        if (val.length == 0) {
            throw new NumberFormatException("Zero length BigInteger");
        } else {
            if (val[0] < 0) {
                this.mag = makePositive(val);
                this.signum = -1;
            } else {
                this.mag = trustedStripLeadingZeroInts(val);
                this.signum = this.mag.length == 0 ? 0 : 1;
            }

            if (this.mag.length >= 67108864) {
                this.checkRange();
            }

        }
    }

    public BigInteger(int signum, byte[] magnitude, int off, int len) {
        if (signum >= -1 && signum <= 1) {
            Objects.checkFromIndexSize(off, len, magnitude.length);
            this.mag = stripLeadingZeroBytes(magnitude, off, len);
            if (this.mag.length == 0) {
                this.signum = 0;
            } else {
                if (signum == 0) {
                    throw new NumberFormatException("signum-magnitude mismatch");
                }

                this.signum = signum;
            }

            if (this.mag.length >= 67108864) {
                this.checkRange();
            }

        } else {
            throw new NumberFormatException("Invalid signum value");
        }
    }

    public BigInteger(int signum, byte[] magnitude) {
        this(signum, magnitude, 0, magnitude.length);
    }

    private BigInteger(int signum, int[] magnitude) {
        this.mag = stripLeadingZeroInts(magnitude);
        if (signum >= -1 && signum <= 1) {
            if (this.mag.length == 0) {
                this.signum = 0;
            } else {
                if (signum == 0) {
                    throw new NumberFormatException("signum-magnitude mismatch");
                }

                this.signum = signum;
            }

            if (this.mag.length >= 67108864) {
                this.checkRange();
            }

        } else {
            throw new NumberFormatException("Invalid signum value");
        }
    }

    public BigInteger(String val, int radix) {
        int cursor = 0;
        int len = val.length();
        if (radix >= 2 && radix <= 36) {
            if (len == 0) {
                throw new NumberFormatException("Zero length BigInteger");
            } else {
                int sign = 1;
                int index1 = val.lastIndexOf(45);
                int index2 = val.lastIndexOf(43);
                if (index1 >= 0) {
                    if (index1 != 0 || index2 >= 0) {
                        throw new NumberFormatException("Illegal embedded sign character");
                    }

                    sign = -1;
                    cursor = 1;
                } else if (index2 >= 0) {
                    if (index2 != 0) {
                        throw new NumberFormatException("Illegal embedded sign character");
                    }

                    cursor = 1;
                }

                if (cursor == len) {
                    throw new NumberFormatException("Zero length BigInteger");
                } else {
                    while(cursor < len && Character.digit(val.charAt(cursor), radix) == 0) {
                        ++cursor;
                    }

                    if (cursor == len) {
                        this.signum = 0;
                        this.mag = ZERO.mag;
                    } else {
                        int numDigits = len - cursor;
                        this.signum = sign;
                        long numBits = ((long)numDigits * bitsPerDigit[radix] >>> 10) + 1L;
                        if (numBits + 31L >= 4294967296L) {
                            reportOverflow();
                        }

                        int numWords = (int)(numBits + 31L) >>> 5;
                        int[] magnitude = new int[numWords];
                        int firstGroupLen = numDigits % digitsPerInt[radix];
                        if (firstGroupLen == 0) {
                            firstGroupLen = digitsPerInt[radix];
                        }

                        String group = val.substring(cursor, cursor += firstGroupLen);
                        magnitude[numWords - 1] = Integer.parseInt(group, radix);
                        if (magnitude[numWords - 1] < 0) {
                            throw new NumberFormatException("Illegal digit");
                        } else {
                            int superRadix = intRadix[radix];
                            boolean var16 = false;

                            while(cursor < len) {
                                group = val.substring(cursor, cursor += digitsPerInt[radix]);
                                int groupVal = Integer.parseInt(group, radix);
                                if (groupVal < 0) {
                                    throw new NumberFormatException("Illegal digit");
                                }

                                destructiveMulAdd(magnitude, superRadix, groupVal);
                            }

                            this.mag = trustedStripLeadingZeroInts(magnitude);
                            if (this.mag.length >= 67108864) {
                                this.checkRange();
                            }

                        }
                    }
                }
            }
        } else {
            throw new NumberFormatException("Radix out of range");
        }
    }

    BigInteger(char[] val, int sign, int len) {
        int cursor;
        for(cursor = 0; cursor < len && Character.digit(val[cursor], 10) == 0; ++cursor) {
        }

        if (cursor == len) {
            this.signum = 0;
            this.mag = ZERO.mag;
        } else {
            int numDigits = len - cursor;
            this.signum = sign;
            int numWords;
            if (len < 10) {
                numWords = 1;
            } else {
                long numBits = ((long)numDigits * bitsPerDigit[10] >>> 10) + 1L;
                if (numBits + 31L >= 4294967296L) {
                    reportOverflow();
                }

                numWords = (int)(numBits + 31L) >>> 5;
            }

            int[] magnitude = new int[numWords];
            int firstGroupLen = numDigits % digitsPerInt[10];
            if (firstGroupLen == 0) {
                firstGroupLen = digitsPerInt[10];
            }

            magnitude[numWords - 1] = this.parseInt(val, cursor, cursor += firstGroupLen);

            while(cursor < len) {
                int groupVal = this.parseInt(val, cursor, cursor += digitsPerInt[10]);
                destructiveMulAdd(magnitude, intRadix[10], groupVal);
            }

            this.mag = trustedStripLeadingZeroInts(magnitude);
            if (this.mag.length >= 67108864) {
                this.checkRange();
            }

        }
    }

    private int parseInt(char[] source, int start, int end) {
        int result = Character.digit(source[start++], 10);
        if (result == -1) {
            throw new NumberFormatException(new String(source));
        } else {
            for(int index = start; index < end; ++index) {
                int nextVal = Character.digit(source[index], 10);
                if (nextVal == -1) {
                    throw new NumberFormatException(new String(source));
                }

                result = 10 * result + nextVal;
            }

            return result;
        }
    }

    private static void destructiveMulAdd(int[] x, int y, int z) {
        long ylong = (long)y & 4294967295L;
        long zlong = (long)z & 4294967295L;
        int len = x.length;
        long product = 0L;
        long carry = 0L;

        for(int i = len - 1; i >= 0; --i) {
            product = ylong * ((long)x[i] & 4294967295L) + carry;
            x[i] = (int)product;
            carry = product >>> 32;
        }

        long sum = ((long)x[len - 1] & 4294967295L) + zlong;
        x[len - 1] = (int)sum;
        carry = sum >>> 32;

        for(int i = len - 2; i >= 0; --i) {
            sum = ((long)x[i] & 4294967295L) + carry;
            x[i] = (int)sum;
            carry = sum >>> 32;
        }

    }

    public BigInteger(String val) {
        this((String)val, 10);
    }

    public BigInteger(int numBits, Random rnd) {
        this(1, (byte[])randomBits(numBits, rnd));
    }

    private static byte[] randomBits(int numBits, Random rnd) {
        if (numBits < 0) {
            throw new IllegalArgumentException("numBits must be non-negative");
        } else {
            int numBytes = (int)(((long)numBits + 7L) / 8L);
            byte[] randomBits = new byte[numBytes];
            if (numBytes > 0) {
                rnd.nextBytes(randomBits);
                int excessBits = 8 * numBytes - numBits;
                randomBits[0] = (byte)(randomBits[0] & (1 << 8 - excessBits) - 1);
            }

            return randomBits;
        }
    }

    public BigInteger(int bitLength, int certainty, Random rnd) {
        if (bitLength < 2) {
            throw new ArithmeticException("bitLength < 2");
        } else {
            BigInteger prime = bitLength < 95 ? smallPrime(bitLength, certainty, rnd) : largePrime(bitLength, certainty, rnd);
            this.signum = 1;
            this.mag = prime.mag;
        }
    }

    public static BigInteger probablePrime(int bitLength, Random rnd) {
        if (bitLength < 2) {
            throw new ArithmeticException("bitLength < 2");
        } else {
            return bitLength < 95 ? smallPrime(bitLength, 100, rnd) : largePrime(bitLength, 100, rnd);
        }
    }

    private static BigInteger smallPrime(int bitLength, int certainty, Random rnd) {
        int magLen = bitLength + 31 >>> 5;
        int[] temp = new int[magLen];
        int highBit = 1 << (bitLength + 31 & 31);
        int highMask = (highBit << 1) - 1;

        BigInteger p;
        do {
            long r;
            do {
                for(int i = 0; i < magLen; ++i) {
                    temp[i] = rnd.nextInt();
                }

                temp[0] = temp[0] & highMask | highBit;
                if (bitLength > 2) {
                    temp[magLen - 1] |= 1;
                }

                p = new BigInteger(temp, 1);
                if (bitLength <= 6) {
                    break;
                }

                r = p.remainder(SMALL_PRIME_PRODUCT).longValue();
            } while(r % 3L == 0L || r % 5L == 0L || r % 7L == 0L || r % 11L == 0L || r % 13L == 0L || r % 17L == 0L || r % 19L == 0L || r % 23L == 0L || r % 29L == 0L || r % 31L == 0L || r % 37L == 0L || r % 41L == 0L);

            if (bitLength < 4) {
                return p;
            }
        } while(!p.primeToCertainty(certainty, rnd));

        return p;
    }

    private static BigInteger largePrime(int bitLength, int certainty, Random rnd) {
        BigInteger p = (new BigInteger(bitLength, rnd)).setBit(bitLength - 1);
        int[] var10000 = p.mag;
        int var10001 = p.mag.length - 1;
        var10000[var10001] &= -2;
        int searchLen = getPrimeSearchLen(bitLength);
        BitSieve searchSieve = new BitSieve(p, searchLen);

        BigInteger candidate;
        for(candidate = searchSieve.retrieve(p, certainty, rnd); candidate == null || candidate.bitLength() != bitLength; candidate = searchSieve.retrieve(p, certainty, rnd)) {
            p = p.add(valueOf((long)(2 * searchLen)));
            if (p.bitLength() != bitLength) {
                p = (new BigInteger(bitLength, rnd)).setBit(bitLength - 1);
            }

            var10000 = p.mag;
            var10001 = p.mag.length - 1;
            var10000[var10001] &= -2;
            searchSieve = new BitSieve(p, searchLen);
        }

        return candidate;
    }

    public BigInteger nextProbablePrime() {
        if (this.signum < 0) {
            throw new ArithmeticException("start < 0: " + this);
        } else if (this.signum != 0 && !this.equals(ONE)) {
            BigInteger result = this.add(ONE);
            if (result.bitLength() < 95) {
                if (!result.testBit(0)) {
                    result = result.add(ONE);
                }

                while(true) {
                    while(true) {
                        if (result.bitLength() > 6) {
                            long r = result.remainder(SMALL_PRIME_PRODUCT).longValue();
                            if (r % 3L == 0L || r % 5L == 0L || r % 7L == 0L || r % 11L == 0L || r % 13L == 0L || r % 17L == 0L || r % 19L == 0L || r % 23L == 0L || r % 29L == 0L || r % 31L == 0L || r % 37L == 0L || r % 41L == 0L) {
                                result = result.add(TWO);
                                continue;
                            }
                        }

                        if (result.bitLength() < 4) {
                            return result;
                        }

                        if (result.primeToCertainty(100, (Random)null)) {
                            return result;
                        }

                        result = result.add(TWO);
                    }
                }
            } else {
                if (result.testBit(0)) {
                    result = result.subtract(ONE);
                }

                int searchLen = getPrimeSearchLen(result.bitLength());

                while(true) {
                    BitSieve searchSieve = new BitSieve(result, searchLen);
                    BigInteger candidate = searchSieve.retrieve(result, 100, (Random)null);
                    if (candidate != null) {
                        return candidate;
                    }

                    result = result.add(valueOf((long)(2 * searchLen)));
                }
            }
        } else {
            return TWO;
        }
    }

    private static int getPrimeSearchLen(int bitLength) {
        if (bitLength > 500000001) {
            throw new ArithmeticException("Prime search implementation restriction on bitLength");
        } else {
            return bitLength / 20 * 64;
        }
    }

    boolean primeToCertainty(int certainty, Random random) {
        int rounds = false;
        int n = (Math.min(certainty, 2147483646) + 1) / 2;
        int sizeInBits = this.bitLength();
        byte rounds;
        int rounds;
        if (sizeInBits < 100) {
            rounds = 50;
            rounds = n < rounds ? n : rounds;
            return this.passesMillerRabin(rounds, random);
        } else {
            if (sizeInBits < 256) {
                rounds = 27;
            } else if (sizeInBits < 512) {
                rounds = 15;
            } else if (sizeInBits < 768) {
                rounds = 8;
            } else if (sizeInBits < 1024) {
                rounds = 4;
            } else {
                rounds = 2;
            }

            rounds = n < rounds ? n : rounds;
            return this.passesMillerRabin(rounds, random) && this.passesLucasLehmer();
        }
    }

    private boolean passesLucasLehmer() {
        BigInteger thisPlusOne = this.add(ONE);

        int d;
        for(d = 5; jacobiSymbol(d, this) != -1; d = d < 0 ? Math.abs(d) + 2 : -(d + 2)) {
        }

        BigInteger u = lucasLehmerSequence(d, thisPlusOne, this);
        return u.mod(this).equals(ZERO);
    }

    private static int jacobiSymbol(int p, BigInteger n) {
        if (p == 0) {
            return 0;
        } else {
            int j = 1;
            int u = n.mag[n.mag.length - 1];
            int t;
            if (p < 0) {
                p = -p;
                t = u & 7;
                if (t == 3 || t == 7) {
                    j = -j;
                }
            }

            while((p & 3) == 0) {
                p >>= 2;
            }

            if ((p & 1) == 0) {
                p >>= 1;
                if (((u ^ u >> 1) & 2) != 0) {
                    j = -j;
                }
            }

            if (p == 1) {
                return j;
            } else {
                if ((p & u & 2) != 0) {
                    j = -j;
                }

                for(u = n.mod(valueOf((long)p)).intValue(); u != 0; u %= t) {
                    while((u & 3) == 0) {
                        u >>= 2;
                    }

                    if ((u & 1) == 0) {
                        u >>= 1;
                        if (((p ^ p >> 1) & 2) != 0) {
                            j = -j;
                        }
                    }

                    if (u == 1) {
                        return j;
                    }

                    assert u < p;

                    t = u;
                    u = p;
                    p = t;
                    if ((u & t & 2) != 0) {
                        j = -j;
                    }
                }

                return 0;
            }
        }
    }

    private static BigInteger lucasLehmerSequence(int z, BigInteger k, BigInteger n) {
        BigInteger d = valueOf((long)z);
        BigInteger u = ONE;
        BigInteger v = ONE;

        for(int i = k.bitLength() - 2; i >= 0; --i) {
            BigInteger u2 = u.multiply(v).mod(n);
            BigInteger v2 = v.square().add(d.multiply(u.square())).mod(n);
            if (v2.testBit(0)) {
                v2 = v2.subtract(n);
            }

            v2 = v2.shiftRight(1);
            u = u2;
            v = v2;
            if (k.testBit(i)) {
                u2 = u2.add(v2).mod(n);
                if (u2.testBit(0)) {
                    u2 = u2.subtract(n);
                }

                u2 = u2.shiftRight(1);
                v2 = v2.add(d.multiply(u)).mod(n);
                if (v2.testBit(0)) {
                    v2 = v2.subtract(n);
                }

                v2 = v2.shiftRight(1);
                u = u2;
                v = v2;
            }
        }

        return u;
    }

    private boolean passesMillerRabin(int iterations, Random rnd) {
        BigInteger thisMinusOne = this.subtract(ONE);
        int a = thisMinusOne.getLowestSetBit();
        BigInteger m = thisMinusOne.shiftRight(a);
        if (rnd == null) {
            rnd = ThreadLocalRandom.current();
        }

        for(int i = 0; i < iterations; ++i) {
            BigInteger b;
            do {
                do {
                    b = new BigInteger(this.bitLength(), (Random)rnd);
                } while(b.compareTo(ONE) <= 0);
            } while(b.compareTo(this) >= 0);

            int j = 0;

            for(BigInteger z = b.modPow(m, this); (j != 0 || !z.equals(ONE)) && !z.equals(thisMinusOne); z = z.modPow(TWO, this)) {
                if (j > 0 && z.equals(ONE)) {
                    return false;
                }

                ++j;
                if (j == a) {
                    return false;
                }
            }
        }

        return true;
    }

    BigInteger(int[] magnitude, int signum) {
        this.signum = magnitude.length == 0 ? 0 : signum;
        this.mag = magnitude;
        if (this.mag.length >= 67108864) {
            this.checkRange();
        }

    }

    private BigInteger(byte[] magnitude, int signum) {
        this.signum = magnitude.length == 0 ? 0 : signum;
        this.mag = stripLeadingZeroBytes(magnitude, 0, magnitude.length);
        if (this.mag.length >= 67108864) {
            this.checkRange();
        }

    }

    private void checkRange() {
        if (this.mag.length > 67108864 || this.mag.length == 67108864 && this.mag[0] < 0) {
            reportOverflow();
        }

    }

    private static void reportOverflow() {
        throw new ArithmeticException("BigInteger would overflow supported range");
    }

    public static BigInteger valueOf(long val) {
        if (val == 0L) {
            return ZERO;
        } else if (val > 0L && val <= 16L) {
            return posConst[(int)val];
        } else {
            return val < 0L && val >= -16L ? negConst[(int)(-val)] : new BigInteger(val);
        }
    }

    private BigInteger(long val) {
        if (val < 0L) {
            val = -val;
            this.signum = -1;
        } else {
            this.signum = 1;
        }

        int highWord = (int)(val >>> 32);
        if (highWord == 0) {
            this.mag = new int[1];
            this.mag[0] = (int)val;
        } else {
            this.mag = new int[2];
            this.mag[0] = highWord;
            this.mag[1] = (int)val;
        }

    }

    private static BigInteger valueOf(int[] val) {
        return val[0] > 0 ? new BigInteger(val, 1) : new BigInteger(val);
    }

    public BigInteger add(BigInteger val) {
        if (val.signum == 0) {
            return this;
        } else if (this.signum == 0) {
            return val;
        } else if (val.signum == this.signum) {
            return new BigInteger(add(this.mag, val.mag), this.signum);
        } else {
            int cmp = this.compareMagnitude(val);
            if (cmp == 0) {
                return ZERO;
            } else {
                int[] resultMag = cmp > 0 ? subtract(this.mag, val.mag) : subtract(val.mag, this.mag);
                resultMag = trustedStripLeadingZeroInts(resultMag);
                return new BigInteger(resultMag, cmp == this.signum ? 1 : -1);
            }
        }
    }

    BigInteger add(long val) {
        if (val == 0L) {
            return this;
        } else if (this.signum == 0) {
            return valueOf(val);
        } else if (Long.signum(val) == this.signum) {
            return new BigInteger(add(this.mag, Math.abs(val)), this.signum);
        } else {
            int cmp = this.compareMagnitude(val);
            if (cmp == 0) {
                return ZERO;
            } else {
                int[] resultMag = cmp > 0 ? subtract(this.mag, Math.abs(val)) : subtract(Math.abs(val), this.mag);
                resultMag = trustedStripLeadingZeroInts(resultMag);
                return new BigInteger(resultMag, cmp == this.signum ? 1 : -1);
            }
        }
    }

    private static int[] add(int[] x, long val) {
        long sum = 0L;
        int xIndex = x.length;
        int highWord = (int)(val >>> 32);
        int[] result;
        if (highWord == 0) {
            result = new int[xIndex];
            --xIndex;
            sum = ((long)x[xIndex] & 4294967295L) + val;
            result[xIndex] = (int)sum;
        } else {
            if (xIndex == 1) {
                result = new int[2];
                sum = val + ((long)x[0] & 4294967295L);
                result[1] = (int)sum;
                result[0] = (int)(sum >>> 32);
                return result;
            }

            result = new int[xIndex];
            --xIndex;
            sum = ((long)x[xIndex] & 4294967295L) + (val & 4294967295L);
            result[xIndex] = (int)sum;
            --xIndex;
            sum = ((long)x[xIndex] & 4294967295L) + ((long)highWord & 4294967295L) + (sum >>> 32);
            result[xIndex] = (int)sum;
        }

        boolean carry;
        for(carry = sum >>> 32 != 0L; xIndex > 0 && carry; carry = (result[xIndex] = x[xIndex] + 1) == 0) {
            --xIndex;
        }

        while(xIndex > 0) {
            --xIndex;
            result[xIndex] = x[xIndex];
        }

        if (carry) {
            int[] bigger = new int[result.length + 1];
            System.arraycopy(result, 0, bigger, 1, result.length);
            bigger[0] = 1;
            return bigger;
        } else {
            return result;
        }
    }

    private static int[] add(int[] x, int[] y) {
        if (x.length < y.length) {
            int[] tmp = x;
            x = y;
            y = tmp;
        }

        int xIndex = x.length;
        int yIndex = y.length;
        int[] result = new int[xIndex];
        long sum = 0L;
        if (yIndex == 1) {
            --xIndex;
            sum = ((long)x[xIndex] & 4294967295L) + ((long)y[0] & 4294967295L);
            result[xIndex] = (int)sum;
        } else {
            while(yIndex > 0) {
                --xIndex;
                long var10000 = (long)x[xIndex] & 4294967295L;
                --yIndex;
                sum = var10000 + ((long)y[yIndex] & 4294967295L) + (sum >>> 32);
                result[xIndex] = (int)sum;
            }
        }

        boolean carry;
        for(carry = sum >>> 32 != 0L; xIndex > 0 && carry; carry = (result[xIndex] = x[xIndex] + 1) == 0) {
            --xIndex;
        }

        while(xIndex > 0) {
            --xIndex;
            result[xIndex] = x[xIndex];
        }

        if (carry) {
            int[] bigger = new int[result.length + 1];
            System.arraycopy(result, 0, bigger, 1, result.length);
            bigger[0] = 1;
            return bigger;
        } else {
            return result;
        }
    }

    private static int[] subtract(long val, int[] little) {
        int highWord = (int)(val >>> 32);
        int[] result;
        if (highWord == 0) {
            result = new int[]{(int)(val - ((long)little[0] & 4294967295L))};
            return result;
        } else {
            result = new int[2];
            long difference;
            if (little.length == 1) {
                difference = ((long)((int)val) & 4294967295L) - ((long)little[0] & 4294967295L);
                result[1] = (int)difference;
                boolean borrow = difference >> 32 != 0L;
                if (borrow) {
                    result[0] = highWord - 1;
                } else {
                    result[0] = highWord;
                }

                return result;
            } else {
                difference = ((long)((int)val) & 4294967295L) - ((long)little[1] & 4294967295L);
                result[1] = (int)difference;
                difference = ((long)highWord & 4294967295L) - ((long)little[0] & 4294967295L) + (difference >> 32);
                result[0] = (int)difference;
                return result;
            }
        }
    }

    private static int[] subtract(int[] big, long val) {
        int highWord = (int)(val >>> 32);
        int bigIndex = big.length;
        int[] result = new int[bigIndex];
        long difference = 0L;
        if (highWord == 0) {
            --bigIndex;
            difference = ((long)big[bigIndex] & 4294967295L) - val;
            result[bigIndex] = (int)difference;
        } else {
            --bigIndex;
            difference = ((long)big[bigIndex] & 4294967295L) - (val & 4294967295L);
            result[bigIndex] = (int)difference;
            --bigIndex;
            difference = ((long)big[bigIndex] & 4294967295L) - ((long)highWord & 4294967295L) + (difference >> 32);
            result[bigIndex] = (int)difference;
        }

        for(boolean borrow = difference >> 32 != 0L; bigIndex > 0 && borrow; borrow = (result[bigIndex] = big[bigIndex] - 1) == -1) {
            --bigIndex;
        }

        while(bigIndex > 0) {
            --bigIndex;
            result[bigIndex] = big[bigIndex];
        }

        return result;
    }

    public BigInteger subtract(BigInteger val) {
        if (val.signum == 0) {
            return this;
        } else if (this.signum == 0) {
            return val.negate();
        } else if (val.signum != this.signum) {
            return new BigInteger(add(this.mag, val.mag), this.signum);
        } else {
            int cmp = this.compareMagnitude(val);
            if (cmp == 0) {
                return ZERO;
            } else {
                int[] resultMag = cmp > 0 ? subtract(this.mag, val.mag) : subtract(val.mag, this.mag);
                resultMag = trustedStripLeadingZeroInts(resultMag);
                return new BigInteger(resultMag, cmp == this.signum ? 1 : -1);
            }
        }
    }

    private static int[] subtract(int[] big, int[] little) {
        int bigIndex = big.length;
        int[] result = new int[bigIndex];
        int littleIndex = little.length;

        long difference;
        for(difference = 0L; littleIndex > 0; result[bigIndex] = (int)difference) {
            --bigIndex;
            long var10000 = (long)big[bigIndex] & 4294967295L;
            --littleIndex;
            difference = var10000 - ((long)little[littleIndex] & 4294967295L) + (difference >> 32);
        }

        for(boolean borrow = difference >> 32 != 0L; bigIndex > 0 && borrow; borrow = (result[bigIndex] = big[bigIndex] - 1) == -1) {
            --bigIndex;
        }

        while(bigIndex > 0) {
            --bigIndex;
            result[bigIndex] = big[bigIndex];
        }

        return result;
    }

    public BigInteger multiply(BigInteger val) {
        return this.multiply(val, false);
    }

    private BigInteger multiply(BigInteger val, boolean isRecursion) {
        if (val.signum != 0 && this.signum != 0) {
            int xlen = this.mag.length;
            if (val == this && xlen > 20) {
                return this.square();
            } else {
                int ylen = val.mag.length;
                if (xlen >= 80 && ylen >= 80) {
                    if (xlen < 240 && ylen < 240) {
                        return multiplyKaratsuba(this, val);
                    } else {
                        if (!isRecursion && (long)(bitLength(this.mag, this.mag.length) + bitLength(val.mag, val.mag.length)) > 2147483648L) {
                            reportOverflow();
                        }

                        return multiplyToomCook3(this, val);
                    }
                } else {
                    int resultSign = this.signum == val.signum ? 1 : -1;
                    if (val.mag.length == 1) {
                        return multiplyByInt(this.mag, val.mag[0], resultSign);
                    } else if (this.mag.length == 1) {
                        return multiplyByInt(val.mag, this.mag[0], resultSign);
                    } else {
                        int[] result = multiplyToLen(this.mag, xlen, val.mag, ylen, (int[])null);
                        result = trustedStripLeadingZeroInts(result);
                        return new BigInteger(result, resultSign);
                    }
                }
            }
        } else {
            return ZERO;
        }
    }

    private static BigInteger multiplyByInt(int[] x, int y, int sign) {
        if (Integer.bitCount(y) == 1) {
            return new BigInteger(shiftLeft(x, Integer.numberOfTrailingZeros(y)), sign);
        } else {
            int xlen = x.length;
            int[] rmag = new int[xlen + 1];
            long carry = 0L;
            long yl = (long)y & 4294967295L;
            int rstart = rmag.length - 1;

            for(int i = xlen - 1; i >= 0; --i) {
                long product = ((long)x[i] & 4294967295L) * yl + carry;
                rmag[rstart--] = (int)product;
                carry = product >>> 32;
            }

            if (carry == 0L) {
                rmag = Arrays.copyOfRange(rmag, 1, rmag.length);
            } else {
                rmag[rstart] = (int)carry;
            }

            return new BigInteger(rmag, sign);
        }
    }

    BigInteger multiply(long v) {
        if (v != 0L && this.signum != 0) {
            if (v == -9223372036854775808L) {
                return this.multiply(valueOf(v));
            } else {
                int rsign = v > 0L ? this.signum : -this.signum;
                if (v < 0L) {
                    v = -v;
                }

                long dh = v >>> 32;
                long dl = v & 4294967295L;
                int xlen = this.mag.length;
                int[] value = this.mag;
                int[] rmag = dh == 0L ? new int[xlen + 1] : new int[xlen + 2];
                long carry = 0L;
                int rstart = rmag.length - 1;

                int i;
                long product;
                for(i = xlen - 1; i >= 0; --i) {
                    product = ((long)value[i] & 4294967295L) * dl + carry;
                    rmag[rstart--] = (int)product;
                    carry = product >>> 32;
                }

                rmag[rstart] = (int)carry;
                if (dh != 0L) {
                    carry = 0L;
                    rstart = rmag.length - 2;

                    for(i = xlen - 1; i >= 0; --i) {
                        product = ((long)value[i] & 4294967295L) * dh + ((long)rmag[rstart] & 4294967295L) + carry;
                        rmag[rstart--] = (int)product;
                        carry = product >>> 32;
                    }

                    rmag[0] = (int)carry;
                }

                if (carry == 0L) {
                    rmag = Arrays.copyOfRange(rmag, 1, rmag.length);
                }

                return new BigInteger(rmag, rsign);
            }
        } else {
            return ZERO;
        }
    }

    private static int[] multiplyToLen(int[] x, int xlen, int[] y, int ylen, int[] z) {
        multiplyToLenCheck(x, xlen);
        multiplyToLenCheck(y, ylen);
        return implMultiplyToLen(x, xlen, y, ylen, z);
    }

    @HotSpotIntrinsicCandidate
    private static int[] implMultiplyToLen(int[] x, int xlen, int[] y, int ylen, int[] z) {
        int xstart = xlen - 1;
        int ystart = ylen - 1;
        if (z == null || z.length < xlen + ylen) {
            z = new int[xlen + ylen];
        }

        long carry = 0L;
        int i = ystart;

        int j;
        for(j = ystart + 1 + xstart; i >= 0; --j) {
            long product = ((long)y[i] & 4294967295L) * ((long)x[xstart] & 4294967295L) + carry;
            z[j] = (int)product;
            carry = product >>> 32;
            --i;
        }

        z[xstart] = (int)carry;

        for(i = xstart - 1; i >= 0; --i) {
            carry = 0L;
            j = ystart;

            for(int k = ystart + 1 + i; j >= 0; --k) {
                long product = ((long)y[j] & 4294967295L) * ((long)x[i] & 4294967295L) + ((long)z[k] & 4294967295L) + carry;
                z[k] = (int)product;
                carry = product >>> 32;
                --j;
            }

            z[i] = (int)carry;
        }

        return z;
    }

    private static void multiplyToLenCheck(int[] array, int length) {
        if (length > 0) {
            Objects.requireNonNull(array);
            if (length > array.length) {
                throw new ArrayIndexOutOfBoundsException(length - 1);
            }
        }
    }

    private static BigInteger multiplyKaratsuba(BigInteger x, BigInteger y) {
        int xlen = x.mag.length;
        int ylen = y.mag.length;
        int half = (Math.max(xlen, ylen) + 1) / 2;
        BigInteger xl = x.getLower(half);
        BigInteger xh = x.getUpper(half);
        BigInteger yl = y.getLower(half);
        BigInteger yh = y.getUpper(half);
        BigInteger p1 = xh.multiply(yh);
        BigInteger p2 = xl.multiply(yl);
        BigInteger p3 = xh.add(xl).multiply(yh.add(yl));
        BigInteger result = p1.shiftLeft(32 * half).add(p3.subtract(p1).subtract(p2)).shiftLeft(32 * half).add(p2);
        return x.signum != y.signum ? result.negate() : result;
    }

    private static BigInteger multiplyToomCook3(BigInteger a, BigInteger b) {
        int alen = a.mag.length;
        int blen = b.mag.length;
        int largest = Math.max(alen, blen);
        int k = (largest + 2) / 3;
        int r = largest - 2 * k;
        BigInteger a2 = a.getToomSlice(k, r, 0, largest);
        BigInteger a1 = a.getToomSlice(k, r, 1, largest);
        BigInteger a0 = a.getToomSlice(k, r, 2, largest);
        BigInteger b2 = b.getToomSlice(k, r, 0, largest);
        BigInteger b1 = b.getToomSlice(k, r, 1, largest);
        BigInteger b0 = b.getToomSlice(k, r, 2, largest);
        BigInteger v0 = a0.multiply(b0, true);
        BigInteger da1 = a2.add(a0);
        BigInteger db1 = b2.add(b0);
        BigInteger vm1 = da1.subtract(a1).multiply(db1.subtract(b1), true);
        da1 = da1.add(a1);
        db1 = db1.add(b1);
        BigInteger v1 = da1.multiply(db1, true);
        BigInteger v2 = da1.add(a2).shiftLeft(1).subtract(a0).multiply(db1.add(b2).shiftLeft(1).subtract(b0), true);
        BigInteger vinf = a2.multiply(b2, true);
        BigInteger t2 = v2.subtract(vm1).exactDivideBy3();
        BigInteger tm1 = v1.subtract(vm1).shiftRight(1);
        BigInteger t1 = v1.subtract(v0);
        t2 = t2.subtract(t1).shiftRight(1);
        t1 = t1.subtract(tm1).subtract(vinf);
        t2 = t2.subtract(vinf.shiftLeft(1));
        tm1 = tm1.subtract(t2);
        int ss = k * 32;
        BigInteger result = vinf.shiftLeft(ss).add(t2).shiftLeft(ss).add(t1).shiftLeft(ss).add(tm1).shiftLeft(ss).add(v0);
        return a.signum != b.signum ? result.negate() : result;
    }

    private BigInteger getToomSlice(int lowerSize, int upperSize, int slice, int fullsize) {
        int len = this.mag.length;
        int offset = fullsize - len;
        int start;
        int end;
        if (slice == 0) {
            start = 0 - offset;
            end = upperSize - 1 - offset;
        } else {
            start = upperSize + (slice - 1) * lowerSize - offset;
            end = start + lowerSize - 1;
        }

        if (start < 0) {
            start = 0;
        }

        if (end < 0) {
            return ZERO;
        } else {
            int sliceSize = end - start + 1;
            if (sliceSize <= 0) {
                return ZERO;
            } else if (start == 0 && sliceSize >= len) {
                return this.abs();
            } else {
                int[] intSlice = new int[sliceSize];
                System.arraycopy(this.mag, start, intSlice, 0, sliceSize);
                return new BigInteger(trustedStripLeadingZeroInts(intSlice), 1);
            }
        }
    }

    private BigInteger exactDivideBy3() {
        int len = this.mag.length;
        int[] result = new int[len];
        long borrow = 0L;

        for(int i = len - 1; i >= 0; --i) {
            long x = (long)this.mag[i] & 4294967295L;
            long w = x - borrow;
            if (borrow > x) {
                borrow = 1L;
            } else {
                borrow = 0L;
            }

            long q = w * 2863311531L & 4294967295L;
            result[i] = (int)q;
            if (q >= 1431655766L) {
                ++borrow;
                if (q >= 2863311531L) {
                    ++borrow;
                }
            }
        }

        result = trustedStripLeadingZeroInts(result);
        return new BigInteger(result, this.signum);
    }

    private BigInteger getLower(int n) {
        int len = this.mag.length;
        if (len <= n) {
            return this.abs();
        } else {
            int[] lowerInts = new int[n];
            System.arraycopy(this.mag, len - n, lowerInts, 0, n);
            return new BigInteger(trustedStripLeadingZeroInts(lowerInts), 1);
        }
    }

    private BigInteger getUpper(int n) {
        int len = this.mag.length;
        if (len <= n) {
            return ZERO;
        } else {
            int upperLen = len - n;
            int[] upperInts = new int[upperLen];
            System.arraycopy(this.mag, 0, upperInts, 0, upperLen);
            return new BigInteger(trustedStripLeadingZeroInts(upperInts), 1);
        }
    }

    private BigInteger square() {
        return this.square(false);
    }

    private BigInteger square(boolean isRecursion) {
        if (this.signum == 0) {
            return ZERO;
        } else {
            int len = this.mag.length;
            if (len < 128) {
                int[] z = squareToLen(this.mag, len, (int[])null);
                return new BigInteger(trustedStripLeadingZeroInts(z), 1);
            } else if (len < 216) {
                return this.squareKaratsuba();
            } else {
                if (!isRecursion && (long)bitLength(this.mag, this.mag.length) > 1073741824L) {
                    reportOverflow();
                }

                return this.squareToomCook3();
            }
        }
    }

    private static final int[] squareToLen(int[] x, int len, int[] z) {
        int zlen = len << 1;
        if (z == null || z.length < zlen) {
            z = new int[zlen];
        }

        implSquareToLenChecks(x, len, z, zlen);
        return implSquareToLen(x, len, z, zlen);
    }

    private static void implSquareToLenChecks(int[] x, int len, int[] z, int zlen) throws RuntimeException {
        if (len < 1) {
            throw new IllegalArgumentException("invalid input length: " + len);
        } else if (len > x.length) {
            throw new IllegalArgumentException("input length out of bound: " + len + " > " + x.length);
        } else if (len * 2 > z.length) {
            throw new IllegalArgumentException("input length out of bound: " + len * 2 + " > " + z.length);
        } else if (zlen < 1) {
            throw new IllegalArgumentException("invalid input length: " + zlen);
        } else if (zlen > z.length) {
            throw new IllegalArgumentException("input length out of bound: " + len + " > " + z.length);
        }
    }

    @HotSpotIntrinsicCandidate
    private static final int[] implSquareToLen(int[] x, int len, int[] z, int zlen) {
        int lastProductLowWord = 0;
        int i = 0;

        int offset;
        for(offset = 0; i < len; ++i) {
            long piece = (long)x[i] & 4294967295L;
            long product = piece * piece;
            z[offset++] = lastProductLowWord << 31 | (int)(product >>> 33);
            z[offset++] = (int)(product >>> 1);
            lastProductLowWord = (int)product;
        }

        i = len;

        for(offset = 1; i > 0; offset += 2) {
            int t = x[i - 1];
            t = mulAdd(z, x, offset, i - 1, t);
            addOne(z, offset - 1, i, t);
            --i;
        }

        primitiveLeftShift(z, zlen, 1);
        z[zlen - 1] |= x[len - 1] & 1;
        return z;
    }

    private BigInteger squareKaratsuba() {
        int half = (this.mag.length + 1) / 2;
        BigInteger xl = this.getLower(half);
        BigInteger xh = this.getUpper(half);
        BigInteger xhs = xh.square();
        BigInteger xls = xl.square();
        return xhs.shiftLeft(half * 32).add(xl.add(xh).square().subtract(xhs.add(xls))).shiftLeft(half * 32).add(xls);
    }

    private BigInteger squareToomCook3() {
        int len = this.mag.length;
        int k = (len + 2) / 3;
        int r = len - 2 * k;
        BigInteger a2 = this.getToomSlice(k, r, 0, len);
        BigInteger a1 = this.getToomSlice(k, r, 1, len);
        BigInteger a0 = this.getToomSlice(k, r, 2, len);
        BigInteger v0 = a0.square(true);
        BigInteger da1 = a2.add(a0);
        BigInteger vm1 = da1.subtract(a1).square(true);
        da1 = da1.add(a1);
        BigInteger v1 = da1.square(true);
        BigInteger vinf = a2.square(true);
        BigInteger v2 = da1.add(a2).shiftLeft(1).subtract(a0).square(true);
        BigInteger t2 = v2.subtract(vm1).exactDivideBy3();
        BigInteger tm1 = v1.subtract(vm1).shiftRight(1);
        BigInteger t1 = v1.subtract(v0);
        t2 = t2.subtract(t1).shiftRight(1);
        t1 = t1.subtract(tm1).subtract(vinf);
        t2 = t2.subtract(vinf.shiftLeft(1));
        tm1 = tm1.subtract(t2);
        int ss = k * 32;
        return vinf.shiftLeft(ss).add(t2).shiftLeft(ss).add(t1).shiftLeft(ss).add(tm1).shiftLeft(ss).add(v0);
    }

    public BigInteger divide(BigInteger val) {
        return val.mag.length >= 80 && this.mag.length - val.mag.length >= 40 ? this.divideBurnikelZiegler(val) : this.divideKnuth(val);
    }

    private BigInteger divideKnuth(BigInteger val) {
        MutableBigInteger q = new MutableBigInteger();
        MutableBigInteger a = new MutableBigInteger(this.mag);
        MutableBigInteger b = new MutableBigInteger(val.mag);
        a.divideKnuth(b, q, false);
        return q.toBigInteger(this.signum * val.signum);
    }

    public BigInteger[] divideAndRemainder(BigInteger val) {
        return val.mag.length >= 80 && this.mag.length - val.mag.length >= 40 ? this.divideAndRemainderBurnikelZiegler(val) : this.divideAndRemainderKnuth(val);
    }

    private BigInteger[] divideAndRemainderKnuth(BigInteger val) {
        BigInteger[] result = new BigInteger[2];
        MutableBigInteger q = new MutableBigInteger();
        MutableBigInteger a = new MutableBigInteger(this.mag);
        MutableBigInteger b = new MutableBigInteger(val.mag);
        MutableBigInteger r = a.divideKnuth(b, q);
        result[0] = q.toBigInteger(this.signum == val.signum ? 1 : -1);
        result[1] = r.toBigInteger(this.signum);
        return result;
    }

    public BigInteger remainder(BigInteger val) {
        return val.mag.length >= 80 && this.mag.length - val.mag.length >= 40 ? this.remainderBurnikelZiegler(val) : this.remainderKnuth(val);
    }

    private BigInteger remainderKnuth(BigInteger val) {
        MutableBigInteger q = new MutableBigInteger();
        MutableBigInteger a = new MutableBigInteger(this.mag);
        MutableBigInteger b = new MutableBigInteger(val.mag);
        return a.divideKnuth(b, q).toBigInteger(this.signum);
    }

    private BigInteger divideBurnikelZiegler(BigInteger val) {
        return this.divideAndRemainderBurnikelZiegler(val)[0];
    }

    private BigInteger remainderBurnikelZiegler(BigInteger val) {
        return this.divideAndRemainderBurnikelZiegler(val)[1];
    }

    private BigInteger[] divideAndRemainderBurnikelZiegler(BigInteger val) {
        MutableBigInteger q = new MutableBigInteger();
        MutableBigInteger r = (new MutableBigInteger(this)).divideAndRemainderBurnikelZiegler(new MutableBigInteger(val), q);
        BigInteger qBigInt = q.isZero() ? ZERO : q.toBigInteger(this.signum * val.signum);
        BigInteger rBigInt = r.isZero() ? ZERO : r.toBigInteger(this.signum);
        return new BigInteger[]{qBigInt, rBigInt};
    }

    public BigInteger pow(int exponent) {
        if (exponent < 0) {
            throw new ArithmeticException("Negative exponent");
        } else if (this.signum == 0) {
            return exponent == 0 ? ONE : this;
        } else {
            BigInteger partToSquare = this.abs();
            int powersOfTwo = partToSquare.getLowestSetBit();
            long bitsToShiftLong = (long)powersOfTwo * (long)exponent;
            if (bitsToShiftLong > 2147483647L) {
                reportOverflow();
            }

            int bitsToShift = (int)bitsToShiftLong;
            int remainingBits;
            if (powersOfTwo > 0) {
                partToSquare = partToSquare.shiftRight(powersOfTwo);
                remainingBits = partToSquare.bitLength();
                if (remainingBits == 1) {
                    if (this.signum < 0 && (exponent & 1) == 1) {
                        return NEGATIVE_ONE.shiftLeft(bitsToShift);
                    }

                    return ONE.shiftLeft(bitsToShift);
                }
            } else {
                remainingBits = partToSquare.bitLength();
                if (remainingBits == 1) {
                    if (this.signum < 0 && (exponent & 1) == 1) {
                        return NEGATIVE_ONE;
                    }

                    return ONE;
                }
            }

            long scaleFactor = (long)remainingBits * (long)exponent;
            if (partToSquare.mag.length == 1 && scaleFactor <= 62L) {
                int newSign = this.signum < 0 && (exponent & 1) == 1 ? -1 : 1;
                long result = 1L;
                long baseToPow2 = (long)partToSquare.mag[0] & 4294967295L;
                int workingExponent = exponent;

                while(workingExponent != 0) {
                    if ((workingExponent & 1) == 1) {
                        result *= baseToPow2;
                    }

                    if ((workingExponent >>>= 1) != 0) {
                        baseToPow2 *= baseToPow2;
                    }
                }

                if (powersOfTwo > 0) {
                    if ((long)bitsToShift + scaleFactor <= 62L) {
                        return valueOf((result << bitsToShift) * (long)newSign);
                    } else {
                        return valueOf(result * (long)newSign).shiftLeft(bitsToShift);
                    }
                } else {
                    return valueOf(result * (long)newSign);
                }
            } else {
                if ((long)this.bitLength() * (long)exponent / 32L > 67108864L) {
                    reportOverflow();
                }

                BigInteger answer = ONE;
                int workingExponent = exponent;

                while(workingExponent != 0) {
                    if ((workingExponent & 1) == 1) {
                        answer = answer.multiply(partToSquare);
                    }

                    if ((workingExponent >>>= 1) != 0) {
                        partToSquare = partToSquare.square();
                    }
                }

                if (powersOfTwo > 0) {
                    answer = answer.shiftLeft(bitsToShift);
                }

                if (this.signum < 0 && (exponent & 1) == 1) {
                    return answer.negate();
                } else {
                    return answer;
                }
            }
        }
    }

    public BigInteger sqrt() {
        if (this.signum < 0) {
            throw new ArithmeticException("Negative BigInteger");
        } else {
            return (new MutableBigInteger(this.mag)).sqrt().toBigInteger();
        }
    }

    public BigInteger[] sqrtAndRemainder() {
        BigInteger s = this.sqrt();
        BigInteger r = this.subtract(s.square());

        assert r.compareTo(ZERO) >= 0;

        return new BigInteger[]{s, r};
    }

    public BigInteger gcd(BigInteger val) {
        if (val.signum == 0) {
            return this.abs();
        } else if (this.signum == 0) {
            return val.abs();
        } else {
            MutableBigInteger a = new MutableBigInteger(this);
            MutableBigInteger b = new MutableBigInteger(val);
            MutableBigInteger result = a.hybridGCD(b);
            return result.toBigInteger(1);
        }
    }

    static int bitLengthForInt(int n) {
        return 32 - Integer.numberOfLeadingZeros(n);
    }

    private static int[] leftShift(int[] a, int len, int n) {
        int nInts = n >>> 5;
        int nBits = n & 31;
        int bitsInHighWord = bitLengthForInt(a[0]);
        if (n <= 32 - bitsInHighWord) {
            primitiveLeftShift(a, len, nBits);
            return a;
        } else {
            int[] result;
            if (nBits <= 32 - bitsInHighWord) {
                result = new int[nInts + len];
                System.arraycopy(a, 0, result, 0, len);
                primitiveLeftShift(result, result.length, nBits);
                return result;
            } else {
                result = new int[nInts + len + 1];
                System.arraycopy(a, 0, result, 0, len);
                primitiveRightShift(result, result.length, 32 - nBits);
                return result;
            }
        }
    }

    static void primitiveRightShift(int[] a, int len, int n) {
        int n2 = 32 - n;
        int i = len - 1;

        for(int c = a[i]; i > 0; --i) {
            int b = c;
            c = a[i - 1];
            a[i] = c << n2 | b >>> n;
        }

        a[0] >>>= n;
    }

    static void primitiveLeftShift(int[] a, int len, int n) {
        if (len != 0 && n != 0) {
            int n2 = 32 - n;
            int i = 0;
            int c = a[i];

            for(int m = i + len - 1; i < m; ++i) {
                int b = c;
                c = a[i + 1];
                a[i] = b << n | c >>> n2;
            }

            a[len - 1] <<= n;
        }
    }

    private static int bitLength(int[] val, int len) {
        return len == 0 ? 0 : (len - 1 << 5) + bitLengthForInt(val[0]);
    }

    public BigInteger abs() {
        return this.signum >= 0 ? this : this.negate();
    }

    public BigInteger negate() {
        return new BigInteger(this.mag, -this.signum);
    }

    public int signum() {
        return this.signum;
    }

    public BigInteger mod(BigInteger m) {
        if (m.signum <= 0) {
            throw new ArithmeticException("BigInteger: modulus not positive");
        } else {
            BigInteger result = this.remainder(m);
            return result.signum >= 0 ? result : result.add(m);
        }
    }

    public BigInteger modPow(BigInteger exponent, BigInteger m) {
        if (m.signum <= 0) {
            throw new ArithmeticException("BigInteger: modulus not positive");
        } else if (exponent.signum == 0) {
            return m.equals(ONE) ? ZERO : ONE;
        } else if (this.equals(ONE)) {
            return m.equals(ONE) ? ZERO : ONE;
        } else if (this.equals(ZERO) && exponent.signum >= 0) {
            return ZERO;
        } else if (this.equals(negConst[1]) && !exponent.testBit(0)) {
            return m.equals(ONE) ? ZERO : ONE;
        } else {
            boolean invertResult;
            if (invertResult = exponent.signum < 0) {
                exponent = exponent.negate();
            }

            BigInteger base = this.signum >= 0 && this.compareTo(m) < 0 ? this : this.mod(m);
            BigInteger result;
            if (m.testBit(0)) {
                result = base.oddModPow(exponent, m);
            } else {
                int p = m.getLowestSetBit();
                BigInteger m1 = m.shiftRight(p);
                BigInteger m2 = ONE.shiftLeft(p);
                BigInteger base2 = this.signum >= 0 && this.compareTo(m1) < 0 ? this : this.mod(m1);
                BigInteger a1 = m1.equals(ONE) ? ZERO : base2.oddModPow(exponent, m1);
                BigInteger a2 = base.modPow2(exponent, p);
                BigInteger y1 = m2.modInverse(m1);
                BigInteger y2 = m1.modInverse(m2);
                if (m.mag.length < 33554432) {
                    result = a1.multiply(m2).multiply(y1).add(a2.multiply(m1).multiply(y2)).mod(m);
                } else {
                    MutableBigInteger t1 = new MutableBigInteger();
                    (new MutableBigInteger(a1.multiply(m2))).multiply(new MutableBigInteger(y1), t1);
                    MutableBigInteger t2 = new MutableBigInteger();
                    (new MutableBigInteger(a2.multiply(m1))).multiply(new MutableBigInteger(y2), t2);
                    t1.add(t2);
                    MutableBigInteger q = new MutableBigInteger();
                    result = t1.divide(new MutableBigInteger(m), q).toBigInteger();
                }
            }

            return invertResult ? result.modInverse(m) : result;
        }
    }

    private static int[] montgomeryMultiply(int[] a, int[] b, int[] n, int len, long inv, int[] product) {
        implMontgomeryMultiplyChecks(a, b, n, len, product);
        if (len > 512) {
            product = multiplyToLen(a, len, b, len, product);
            return montReduce(product, n, len, (int)inv);
        } else {
            return implMontgomeryMultiply(a, b, n, len, inv, materialize(product, len));
        }
    }

    private static int[] montgomerySquare(int[] a, int[] n, int len, long inv, int[] product) {
        implMontgomeryMultiplyChecks(a, a, n, len, product);
        if (len > 512) {
            product = squareToLen(a, len, product);
            return montReduce(product, n, len, (int)inv);
        } else {
            return implMontgomerySquare(a, n, len, inv, materialize(product, len));
        }
    }

    private static void implMontgomeryMultiplyChecks(int[] a, int[] b, int[] n, int len, int[] product) throws RuntimeException {
        if (len % 2 != 0) {
            throw new IllegalArgumentException("input array length must be even: " + len);
        } else if (len < 1) {
            throw new IllegalArgumentException("invalid input length: " + len);
        } else if (len > a.length || len > b.length || len > n.length || product != null && len > product.length) {
            throw new IllegalArgumentException("input array length out of bound: " + len);
        }
    }

    private static int[] materialize(int[] z, int len) {
        if (z == null || z.length < len) {
            z = new int[len];
        }

        return z;
    }

    @HotSpotIntrinsicCandidate
    private static int[] implMontgomeryMultiply(int[] a, int[] b, int[] n, int len, long inv, int[] product) {
        product = multiplyToLen(a, len, b, len, product);
        return montReduce(product, n, len, (int)inv);
    }

    @HotSpotIntrinsicCandidate
    private static int[] implMontgomerySquare(int[] a, int[] n, int len, long inv, int[] product) {
        product = squareToLen(a, len, product);
        return montReduce(product, n, len, (int)inv);
    }

    private BigInteger oddModPow(BigInteger y, BigInteger z) {
        if (y.equals(ONE)) {
            return this;
        } else if (this.signum == 0) {
            return ZERO;
        } else {
            int[] base = (int[])this.mag.clone();
            int[] exp = y.mag;
            int[] mod = z.mag;
            int modLen = mod.length;
            if ((modLen & 1) != 0) {
                int[] x = new int[modLen + 1];
                System.arraycopy(mod, 0, x, 1, modLen);
                mod = x;
                ++modLen;
            }

            int wbits = 0;
            int ebits = bitLength(exp, exp.length);
            if (ebits != 17 || exp[0] != 65537) {
                while(ebits > bnExpModThreshTable[wbits]) {
                    ++wbits;
                }
            }

            int tblmask = 1 << wbits;
            int[][] table = new int[tblmask][];

            for(int i = 0; i < tblmask; ++i) {
                table[i] = new int[modLen];
            }

            long n0 = ((long)mod[modLen - 1] & 4294967295L) + (((long)mod[modLen - 2] & 4294967295L) << 32);
            long inv = -MutableBigInteger.inverseMod64(n0);
            int[] a = leftShift(base, base.length, modLen << 5);
            MutableBigInteger q = new MutableBigInteger();
            MutableBigInteger a2 = new MutableBigInteger(a);
            MutableBigInteger b2 = new MutableBigInteger(mod);
            b2.normalize();
            MutableBigInteger r = a2.divide(b2, q);
            table[0] = r.toIntArray();
            int[] t;
            if (table[0].length < modLen) {
                int offset = modLen - table[0].length;
                t = new int[modLen];
                System.arraycopy(table[0], 0, t, offset, table[0].length);
                table[0] = t;
            }

            int[] b = montgomerySquare(table[0], mod, modLen, inv, (int[])null);
            t = Arrays.copyOf(b, modLen);

            int bitpos;
            for(bitpos = 1; bitpos < tblmask; ++bitpos) {
                table[bitpos] = montgomeryMultiply(t, table[bitpos - 1], mod, modLen, inv, (int[])null);
            }

            bitpos = 1 << (ebits - 1 & 31);
            int buf = 0;
            int elen = exp.length;
            int eIndex = 0;

            int multpos;
            for(multpos = 0; multpos <= wbits; ++multpos) {
                buf = buf << 1 | ((exp[eIndex] & bitpos) != 0 ? 1 : 0);
                bitpos >>>= 1;
                if (bitpos == 0) {
                    ++eIndex;
                    bitpos = -2147483648;
                    --elen;
                }
            }

            --ebits;
            boolean isone = true;

            for(multpos = ebits - wbits; (buf & 1) == 0; ++multpos) {
                buf >>>= 1;
            }

            int[] mult = table[buf >>> 1];
            buf = 0;
            if (multpos == ebits) {
                isone = false;
            }

            while(true) {
                --ebits;
                buf <<= 1;
                if (elen != 0) {
                    buf |= (exp[eIndex] & bitpos) != 0 ? 1 : 0;
                    bitpos >>>= 1;
                    if (bitpos == 0) {
                        ++eIndex;
                        bitpos = -2147483648;
                        --elen;
                    }
                }

                if ((buf & tblmask) != 0) {
                    for(multpos = ebits - wbits; (buf & 1) == 0; ++multpos) {
                        buf >>>= 1;
                    }

                    mult = table[buf >>> 1];
                    buf = 0;
                }

                if (ebits == multpos) {
                    if (isone) {
                        b = (int[])mult.clone();
                        isone = false;
                    } else {
                        a = montgomeryMultiply(b, mult, mod, modLen, inv, a);
                        t = a;
                        a = b;
                        b = t;
                    }
                }

                if (ebits == 0) {
                    int[] t2 = new int[2 * modLen];
                    System.arraycopy(b, 0, t2, modLen, modLen);
                    b = montReduce(t2, mod, modLen, (int)inv);
                    t2 = Arrays.copyOf(b, modLen);
                    return new BigInteger(1, t2);
                }

                if (!isone) {
                    a = montgomerySquare(b, mod, modLen, inv, a);
                    t = a;
                    a = b;
                    b = t;
                }
            }
        }
    }

    private static int[] montReduce(int[] n, int[] mod, int mlen, int inv) {
        int c = 0;
        int len = mlen;
        int offset = 0;

        do {
            int nEnd = n[n.length - 1 - offset];
            int carry = mulAdd(n, mod, offset, mlen, inv * nEnd);
            c += addOne(n, offset, mlen, carry);
            ++offset;
            --len;
        } while(len > 0);

        while(c > 0) {
            c += subN(n, mod, mlen);
        }

        while(intArrayCmpToLen(n, mod, mlen) >= 0) {
            subN(n, mod, mlen);
        }

        return n;
    }

    private static int intArrayCmpToLen(int[] arg1, int[] arg2, int len) {
        for(int i = 0; i < len; ++i) {
            long b1 = (long)arg1[i] & 4294967295L;
            long b2 = (long)arg2[i] & 4294967295L;
            if (b1 < b2) {
                return -1;
            }

            if (b1 > b2) {
                return 1;
            }
        }

        return 0;
    }

    private static int subN(int[] a, int[] b, int len) {
        long sum = 0L;

        while(true) {
            --len;
            if (len < 0) {
                return (int)(sum >> 32);
            }

            sum = ((long)a[len] & 4294967295L) - ((long)b[len] & 4294967295L) + (sum >> 32);
            a[len] = (int)sum;
        }
    }

    static int mulAdd(int[] out, int[] in, int offset, int len, int k) {
        implMulAddCheck(out, in, offset, len, k);
        return implMulAdd(out, in, offset, len, k);
    }

    private static void implMulAddCheck(int[] out, int[] in, int offset, int len, int k) {
        if (len > in.length) {
            throw new IllegalArgumentException("input length is out of bound: " + len + " > " + in.length);
        } else if (offset < 0) {
            throw new IllegalArgumentException("input offset is invalid: " + offset);
        } else if (offset > out.length - 1) {
            throw new IllegalArgumentException("input offset is out of bound: " + offset + " > " + (out.length - 1));
        } else if (len > out.length - offset) {
            throw new IllegalArgumentException("input len is out of bound: " + len + " > " + (out.length - offset));
        }
    }

    @HotSpotIntrinsicCandidate
    private static int implMulAdd(int[] out, int[] in, int offset, int len, int k) {
        long kLong = (long)k & 4294967295L;
        long carry = 0L;
        offset = out.length - offset - 1;

        for(int j = len - 1; j >= 0; --j) {
            long product = ((long)in[j] & 4294967295L) * kLong + ((long)out[offset] & 4294967295L) + carry;
            out[offset--] = (int)product;
            carry = product >>> 32;
        }

        return (int)carry;
    }

    static int addOne(int[] a, int offset, int mlen, int carry) {
        offset = a.length - 1 - mlen - offset;
        long t = ((long)a[offset] & 4294967295L) + ((long)carry & 4294967295L);
        a[offset] = (int)t;
        if (t >>> 32 == 0L) {
            return 0;
        } else {
            do {
                --mlen;
                if (mlen < 0) {
                    return 1;
                }

                --offset;
                if (offset < 0) {
                    return 1;
                }

                int var10002 = a[offset]++;
            } while(a[offset] == 0);

            return 0;
        }
    }

    private BigInteger modPow2(BigInteger exponent, int p) {
        BigInteger result = ONE;
        BigInteger baseToPow2 = this.mod2(p);
        int expOffset = 0;
        int limit = exponent.bitLength();
        if (this.testBit(0)) {
            limit = p - 1 < limit ? p - 1 : limit;
        }

        while(expOffset < limit) {
            if (exponent.testBit(expOffset)) {
                result = result.multiply(baseToPow2).mod2(p);
            }

            ++expOffset;
            if (expOffset < limit) {
                baseToPow2 = baseToPow2.square().mod2(p);
            }
        }

        return result;
    }

    private BigInteger mod2(int p) {
        if (this.bitLength() <= p) {
            return this;
        } else {
            int numInts = p + 31 >>> 5;
            int[] mag = new int[numInts];
            System.arraycopy(this.mag, this.mag.length - numInts, mag, 0, numInts);
            int excessBits = (numInts << 5) - p;
            mag[0] = (int)((long)mag[0] & (1L << 32 - excessBits) - 1L);
            return mag[0] == 0 ? new BigInteger(1, mag) : new BigInteger(mag, 1);
        }
    }

    public BigInteger modInverse(BigInteger m) {
        if (m.signum != 1) {
            throw new ArithmeticException("BigInteger: modulus not positive");
        } else if (m.equals(ONE)) {
            return ZERO;
        } else {
            BigInteger modVal = this;
            if (this.signum < 0 || this.compareMagnitude(m) >= 0) {
                modVal = this.mod(m);
            }

            if (modVal.equals(ONE)) {
                return ONE;
            } else {
                MutableBigInteger a = new MutableBigInteger(modVal);
                MutableBigInteger b = new MutableBigInteger(m);
                MutableBigInteger result = a.mutableModInverse(b);
                return result.toBigInteger(1);
            }
        }
    }

    public BigInteger shiftLeft(int n) {
        if (this.signum == 0) {
            return ZERO;
        } else if (n > 0) {
            return new BigInteger(shiftLeft(this.mag, n), this.signum);
        } else {
            return n == 0 ? this : this.shiftRightImpl(-n);
        }
    }

    private static int[] shiftLeft(int[] mag, int n) {
        int nInts = n >>> 5;
        int nBits = n & 31;
        int magLen = mag.length;
        int[] newMag = null;
        int[] newMag;
        if (nBits == 0) {
            newMag = new int[magLen + nInts];
            System.arraycopy(mag, 0, newMag, 0, magLen);
        } else {
            int i = 0;
            int nBits2 = 32 - nBits;
            int highBits = mag[0] >>> nBits2;
            if (highBits != 0) {
                newMag = new int[magLen + nInts + 1];
                newMag[i++] = highBits;
            } else {
                newMag = new int[magLen + nInts];
            }

            int j;
            for(j = 0; j < magLen - 1; newMag[i++] = mag[j++] << nBits | mag[j] >>> nBits2) {
            }

            newMag[i] = mag[j] << nBits;
        }

        return newMag;
    }

    public BigInteger shiftRight(int n) {
        if (this.signum == 0) {
            return ZERO;
        } else if (n > 0) {
            return this.shiftRightImpl(n);
        } else {
            return n == 0 ? this : new BigInteger(shiftLeft(this.mag, -n), this.signum);
        }
    }

    private BigInteger shiftRightImpl(int n) {
        int nInts = n >>> 5;
        int nBits = n & 31;
        int magLen = this.mag.length;
        int[] newMag = null;
        if (nInts >= magLen) {
            return this.signum >= 0 ? ZERO : negConst[1];
        } else {
            int newMagLen;
            int i;
            int nBits2;
            int[] newMag;
            if (nBits == 0) {
                newMagLen = magLen - nInts;
                newMag = Arrays.copyOf(this.mag, newMagLen);
            } else {
                newMagLen = 0;
                i = this.mag[0] >>> nBits;
                if (i != 0) {
                    newMag = new int[magLen - nInts];
                    newMag[newMagLen++] = i;
                } else {
                    newMag = new int[magLen - nInts - 1];
                }

                nBits2 = 32 - nBits;

                for(int j = 0; j < magLen - nInts - 1; newMag[newMagLen++] = this.mag[j++] << nBits2 | this.mag[j] >>> nBits) {
                }
            }

            if (this.signum < 0) {
                boolean onesLost = false;
                i = magLen - 1;

                for(nBits2 = magLen - nInts; i >= nBits2 && !onesLost; --i) {
                    onesLost = this.mag[i] != 0;
                }

                if (!onesLost && nBits != 0) {
                    onesLost = this.mag[magLen - nInts - 1] << 32 - nBits != 0;
                }

                if (onesLost) {
                    newMag = this.javaIncrement(newMag);
                }
            }

            return new BigInteger(newMag, this.signum);
        }
    }

    int[] javaIncrement(int[] val) {
        int lastSum = 0;

        for(int i = val.length - 1; i >= 0 && lastSum == 0; --i) {
            lastSum = ++val[i];
        }

        if (lastSum == 0) {
            val = new int[val.length + 1];
            val[0] = 1;
        }

        return val;
    }

    public BigInteger and(BigInteger val) {
        int[] result = new int[Math.max(this.intLength(), val.intLength())];

        for(int i = 0; i < result.length; ++i) {
            result[i] = this.getInt(result.length - i - 1) & val.getInt(result.length - i - 1);
        }

        return valueOf(result);
    }

    public BigInteger or(BigInteger val) {
        int[] result = new int[Math.max(this.intLength(), val.intLength())];

        for(int i = 0; i < result.length; ++i) {
            result[i] = this.getInt(result.length - i - 1) | val.getInt(result.length - i - 1);
        }

        return valueOf(result);
    }

    public BigInteger xor(BigInteger val) {
        int[] result = new int[Math.max(this.intLength(), val.intLength())];

        for(int i = 0; i < result.length; ++i) {
            result[i] = this.getInt(result.length - i - 1) ^ val.getInt(result.length - i - 1);
        }

        return valueOf(result);
    }

    public BigInteger not() {
        int[] result = new int[this.intLength()];

        for(int i = 0; i < result.length; ++i) {
            result[i] = ~this.getInt(result.length - i - 1);
        }

        return valueOf(result);
    }

    public BigInteger andNot(BigInteger val) {
        int[] result = new int[Math.max(this.intLength(), val.intLength())];

        for(int i = 0; i < result.length; ++i) {
            result[i] = this.getInt(result.length - i - 1) & ~val.getInt(result.length - i - 1);
        }

        return valueOf(result);
    }

    public boolean testBit(int n) {
        if (n < 0) {
            throw new ArithmeticException("Negative bit address");
        } else {
            return (this.getInt(n >>> 5) & 1 << (n & 31)) != 0;
        }
    }

    public BigInteger setBit(int n) {
        if (n < 0) {
            throw new ArithmeticException("Negative bit address");
        } else {
            int intNum = n >>> 5;
            int[] result = new int[Math.max(this.intLength(), intNum + 2)];

            for(int i = 0; i < result.length; ++i) {
                result[result.length - i - 1] = this.getInt(i);
            }

            result[result.length - intNum - 1] |= 1 << (n & 31);
            return valueOf(result);
        }
    }

    public BigInteger clearBit(int n) {
        if (n < 0) {
            throw new ArithmeticException("Negative bit address");
        } else {
            int intNum = n >>> 5;
            int[] result = new int[Math.max(this.intLength(), (n + 1 >>> 5) + 1)];

            for(int i = 0; i < result.length; ++i) {
                result[result.length - i - 1] = this.getInt(i);
            }

            result[result.length - intNum - 1] &= ~(1 << (n & 31));
            return valueOf(result);
        }
    }

    public BigInteger flipBit(int n) {
        if (n < 0) {
            throw new ArithmeticException("Negative bit address");
        } else {
            int intNum = n >>> 5;
            int[] result = new int[Math.max(this.intLength(), intNum + 2)];

            for(int i = 0; i < result.length; ++i) {
                result[result.length - i - 1] = this.getInt(i);
            }

            result[result.length - intNum - 1] ^= 1 << (n & 31);
            return valueOf(result);
        }
    }

    public int getLowestSetBit() {
        int lsb = this.lowestSetBitPlusTwo - 2;
        if (lsb == -2) {
            int lsb = 0;
            if (this.signum == 0) {
                lsb = lsb - 1;
            } else {
                int i;
                int b;
                for(i = 0; (b = this.getInt(i)) == 0; ++i) {
                }

                lsb = lsb + (i << 5) + Integer.numberOfTrailingZeros(b);
            }

            this.lowestSetBitPlusTwo = lsb + 2;
        }

        return lsb;
    }

    public int bitLength() {
        int n = this.bitLengthPlusOne - 1;
        if (n == -1) {
            int[] m = this.mag;
            int len = m.length;
            if (len == 0) {
                n = 0;
            } else {
                int magBitLength = (len - 1 << 5) + bitLengthForInt(this.mag[0]);
                if (this.signum >= 0) {
                    n = magBitLength;
                } else {
                    boolean pow2 = Integer.bitCount(this.mag[0]) == 1;

                    for(int i = 1; i < len && pow2; ++i) {
                        pow2 = this.mag[i] == 0;
                    }

                    n = pow2 ? magBitLength - 1 : magBitLength;
                }
            }

            this.bitLengthPlusOne = n + 1;
        }

        return n;
    }

    public int bitCount() {
        int bc = this.bitCountPlusOne - 1;
        if (bc == -1) {
            bc = 0;

            int magTrailingZeroCount;
            for(magTrailingZeroCount = 0; magTrailingZeroCount < this.mag.length; ++magTrailingZeroCount) {
                bc += Integer.bitCount(this.mag[magTrailingZeroCount]);
            }

            if (this.signum < 0) {
                magTrailingZeroCount = 0;

                int j;
                for(j = this.mag.length - 1; this.mag[j] == 0; --j) {
                    magTrailingZeroCount += 32;
                }

                magTrailingZeroCount += Integer.numberOfTrailingZeros(this.mag[j]);
                bc += magTrailingZeroCount - 1;
            }

            this.bitCountPlusOne = bc + 1;
        }

        return bc;
    }

    public boolean isProbablePrime(int certainty) {
        if (certainty <= 0) {
            return true;
        } else {
            BigInteger w = this.abs();
            if (w.equals(TWO)) {
                return true;
            } else {
                return w.testBit(0) && !w.equals(ONE) ? w.primeToCertainty(certainty, (Random)null) : false;
            }
        }
    }

    public int compareTo(BigInteger val) {
        if (this.signum == val.signum) {
            switch(this.signum) {
            case -1:
                return val.compareMagnitude(this);
            case 1:
                return this.compareMagnitude(val);
            default:
                return 0;
            }
        } else {
            return this.signum > val.signum ? 1 : -1;
        }
    }

    final int compareMagnitude(BigInteger val) {
        int[] m1 = this.mag;
        int len1 = m1.length;
        int[] m2 = val.mag;
        int len2 = m2.length;
        if (len1 < len2) {
            return -1;
        } else if (len1 > len2) {
            return 1;
        } else {
            for(int i = 0; i < len1; ++i) {
                int a = m1[i];
                int b = m2[i];
                if (a != b) {
                    return ((long)a & 4294967295L) < ((long)b & 4294967295L) ? -1 : 1;
                }
            }

            return 0;
        }
    }

    final int compareMagnitude(long val) {
        assert val != -9223372036854775808L;

        int[] m1 = this.mag;
        int len = m1.length;
        if (len > 2) {
            return 1;
        } else {
            if (val < 0L) {
                val = -val;
            }

            int highWord = (int)(val >>> 32);
            int a;
            int b;
            if (highWord == 0) {
                if (len < 1) {
                    return -1;
                } else if (len > 1) {
                    return 1;
                } else {
                    a = m1[0];
                    b = (int)val;
                    if (a != b) {
                        return ((long)a & 4294967295L) < ((long)b & 4294967295L) ? -1 : 1;
                    } else {
                        return 0;
                    }
                }
            } else if (len < 2) {
                return -1;
            } else {
                a = m1[0];
                if (a != highWord) {
                    return ((long)a & 4294967295L) < ((long)highWord & 4294967295L) ? -1 : 1;
                } else {
                    a = m1[1];
                    b = (int)val;
                    if (a != b) {
                        return ((long)a & 4294967295L) < ((long)b & 4294967295L) ? -1 : 1;
                    } else {
                        return 0;
                    }
                }
            }
        }
    }

    public boolean equals(Object x) {
        if (x == this) {
            return true;
        } else if (!(x instanceof BigInteger)) {
            return false;
        } else {
            BigInteger xInt = (BigInteger)x;
            if (xInt.signum != this.signum) {
                return false;
            } else {
                int[] m = this.mag;
                int len = m.length;
                int[] xm = xInt.mag;
                if (len != xm.length) {
                    return false;
                } else {
                    for(int i = 0; i < len; ++i) {
                        if (xm[i] != m[i]) {
                            return false;
                        }
                    }

                    return true;
                }
            }
        }
    }

    public BigInteger min(BigInteger val) {
        return this.compareTo(val) < 0 ? this : val;
    }

    public BigInteger max(BigInteger val) {
        return this.compareTo(val) > 0 ? this : val;
    }

    public int hashCode() {
        int hashCode = 0;

        for(int i = 0; i < this.mag.length; ++i) {
            hashCode = (int)((long)(31 * hashCode) + ((long)this.mag[i] & 4294967295L));
        }

        return hashCode * this.signum;
    }

    public String toString(int radix) {
        if (this.signum == 0) {
            return "0";
        } else {
            if (radix < 2 || radix > 36) {
                radix = 10;
            }

            if (this.mag.length <= 20) {
                return this.smallToString(radix);
            } else {
                StringBuilder sb = new StringBuilder();
                if (this.signum < 0) {
                    toString(this.negate(), sb, radix, 0);
                    sb.insert(0, '-');
                } else {
                    toString(this, sb, radix, 0);
                }

                return sb.toString();
            }
        }
    }

    private String smallToString(int radix) {
        if (this.signum == 0) {
            return "0";
        } else {
            int maxNumDigitGroups = (4 * this.mag.length + 6) / 7;
            String[] digitGroup = new String[maxNumDigitGroups];
            BigInteger tmp = this.abs();

            int numGroups;
            BigInteger q2;
            for(numGroups = 0; tmp.signum != 0; tmp = q2) {
                BigInteger d = longRadix[radix];
                MutableBigInteger q = new MutableBigInteger();
                MutableBigInteger a = new MutableBigInteger(tmp.mag);
                MutableBigInteger b = new MutableBigInteger(d.mag);
                MutableBigInteger r = a.divide(b, q);
                q2 = q.toBigInteger(tmp.signum * d.signum);
                BigInteger r2 = r.toBigInteger(tmp.signum * d.signum);
                digitGroup[numGroups++] = Long.toString(r2.longValue(), radix);
            }

            StringBuilder buf = new StringBuilder(numGroups * digitsPerLong[radix] + 1);
            if (this.signum < 0) {
                buf.append('-');
            }

            buf.append(digitGroup[numGroups - 1]);

            for(int i = numGroups - 2; i >= 0; --i) {
                int numLeadingZeros = digitsPerLong[radix] - digitGroup[i].length();
                if (numLeadingZeros != 0) {
                    buf.append(zeros[numLeadingZeros]);
                }

                buf.append(digitGroup[i]);
            }

            return buf.toString();
        }
    }

    private static void toString(BigInteger u, StringBuilder sb, int radix, int digits) {
        int i;
        if (u.mag.length > 20) {
            int b = u.bitLength();
            i = (int)Math.round(Math.log((double)b * LOG_TWO / logCache[radix]) / LOG_TWO - 1.0D);
            BigInteger v = getRadixConversionCache(radix, i);
            BigInteger[] results = u.divideAndRemainder(v);
            int expectedDigits = 1 << i;
            toString(results[0], sb, radix, digits - expectedDigits);
            toString(results[1], sb, radix, expectedDigits);
        } else {
            String s = u.smallToString(radix);
            if (s.length() < digits && sb.length() > 0) {
                for(i = s.length(); i < digits; ++i) {
                    sb.append('0');
                }
            }

            sb.append(s);
        }
    }

    private static BigInteger getRadixConversionCache(int radix, int exponent) {
        BigInteger[] cacheLine = powerCache[radix];
        if (exponent < cacheLine.length) {
            return cacheLine[exponent];
        } else {
            int oldLength = cacheLine.length;
            cacheLine = (BigInteger[])Arrays.copyOf(cacheLine, exponent + 1);

            for(int i = oldLength; i <= exponent; ++i) {
                cacheLine[i] = cacheLine[i - 1].pow(2);
            }

            BigInteger[][] pc = powerCache;
            if (exponent >= pc[radix].length) {
                pc = (BigInteger[][])pc.clone();
                pc[radix] = cacheLine;
                powerCache = pc;
            }

            return cacheLine[exponent];
        }
    }

    public String toString() {
        return this.toString(10);
    }

    public byte[] toByteArray() {
        int byteLen = this.bitLength() / 8 + 1;
        byte[] byteArray = new byte[byteLen];
        int i = byteLen - 1;
        int bytesCopied = 4;
        int nextInt = 0;

        for(int var6 = 0; i >= 0; --i) {
            if (bytesCopied == 4) {
                nextInt = this.getInt(var6++);
                bytesCopied = 1;
            } else {
                nextInt >>>= 8;
                ++bytesCopied;
            }

            byteArray[i] = (byte)nextInt;
        }

        return byteArray;
    }

    public int intValue() {
        int result = false;
        int result = this.getInt(0);
        return result;
    }

    public long longValue() {
        long result = 0L;

        for(int i = 1; i >= 0; --i) {
            result = (result << 32) + ((long)this.getInt(i) & 4294967295L);
        }

        return result;
    }

    public float floatValue() {
        if (this.signum == 0) {
            return 0.0F;
        } else {
            int exponent = (this.mag.length - 1 << 5) + bitLengthForInt(this.mag[0]) - 1;
            if (exponent < 63) {
                return (float)this.longValue();
            } else if (exponent > 127) {
                return this.signum > 0 ? 1.0F / 0.0 : -1.0F / 0.0;
            } else {
                int shift = exponent - 24;
                int nBits = shift & 31;
                int nBits2 = 32 - nBits;
                int twiceSignifFloor;
                if (nBits == 0) {
                    twiceSignifFloor = this.mag[0];
                } else {
                    twiceSignifFloor = this.mag[0] >>> nBits;
                    if (twiceSignifFloor == 0) {
                        twiceSignifFloor = this.mag[0] << nBits2 | this.mag[1] >>> nBits;
                    }
                }

                int signifFloor = twiceSignifFloor >> 1;
                signifFloor &= 8388607;
                boolean increment = (twiceSignifFloor & 1) != 0 && ((signifFloor & 1) != 0 || this.abs().getLowestSetBit() < shift);
                int signifRounded = increment ? signifFloor + 1 : signifFloor;
                int bits = exponent + 127 << 23;
                bits += signifRounded;
                bits |= this.signum & -2147483648;
                return Float.intBitsToFloat(bits);
            }
        }
    }

    public double doubleValue() {
        if (this.signum == 0) {
            return 0.0D;
        } else {
            int exponent = (this.mag.length - 1 << 5) + bitLengthForInt(this.mag[0]) - 1;
            if (exponent < 63) {
                return (double)this.longValue();
            } else if (exponent > 1023) {
                return this.signum > 0 ? 1.0D / 0.0 : -1.0D / 0.0;
            } else {
                int shift = exponent - 53;
                int nBits = shift & 31;
                int nBits2 = 32 - nBits;
                int highBits;
                int lowBits;
                if (nBits == 0) {
                    highBits = this.mag[0];
                    lowBits = this.mag[1];
                } else {
                    highBits = this.mag[0] >>> nBits;
                    lowBits = this.mag[0] << nBits2 | this.mag[1] >>> nBits;
                    if (highBits == 0) {
                        highBits = lowBits;
                        lowBits = this.mag[1] << nBits2 | this.mag[2] >>> nBits;
                    }
                }

                long twiceSignifFloor = ((long)highBits & 4294967295L) << 32 | (long)lowBits & 4294967295L;
                long signifFloor = twiceSignifFloor >> 1;
                signifFloor &= 4503599627370495L;
                boolean increment = (twiceSignifFloor & 1L) != 0L && ((signifFloor & 1L) != 0L || this.abs().getLowestSetBit() < shift);
                long signifRounded = increment ? signifFloor + 1L : signifFloor;
                long bits = (long)(exponent + 1023) << 52;
                bits += signifRounded;
                bits |= (long)this.signum & -9223372036854775808L;
                return Double.longBitsToDouble(bits);
            }
        }
    }

    private static int[] stripLeadingZeroInts(int[] val) {
        int vlen = val.length;

        int keep;
        for(keep = 0; keep < vlen && val[keep] == 0; ++keep) {
        }

        return Arrays.copyOfRange(val, keep, vlen);
    }

    private static int[] trustedStripLeadingZeroInts(int[] val) {
        int vlen = val.length;

        int keep;
        for(keep = 0; keep < vlen && val[keep] == 0; ++keep) {
        }

        return keep == 0 ? val : Arrays.copyOfRange(val, keep, vlen);
    }

    private static int[] stripLeadingZeroBytes(byte[] a, int off, int len) {
        int indexBound = off + len;

        int keep;
        for(keep = off; keep < indexBound && a[keep] == 0; ++keep) {
        }

        int intLength = indexBound - keep + 3 >>> 2;
        int[] result = new int[intLength];
        int b = indexBound - 1;

        for(int i = intLength - 1; i >= 0; --i) {
            result[i] = a[b--] & 255;
            int bytesRemaining = b - keep + 1;
            int bytesToTransfer = Math.min(3, bytesRemaining);

            for(int j = 8; j <= bytesToTransfer << 3; j += 8) {
                result[i] |= (a[b--] & 255) << j;
            }
        }

        return result;
    }

    private static int[] makePositive(byte[] a, int off, int len) {
        int indexBound = off + len;

        int keep;
        for(keep = off; keep < indexBound && a[keep] == -1; ++keep) {
        }

        int k;
        for(k = keep; k < indexBound && a[k] == 0; ++k) {
        }

        int extraByte = k == indexBound ? 1 : 0;
        int intLength = indexBound - keep + extraByte + 3 >>> 2;
        int[] result = new int[intLength];
        int b = indexBound - 1;

        int i;
        for(i = intLength - 1; i >= 0; --i) {
            result[i] = a[b--] & 255;
            int numBytesToTransfer = Math.min(3, b - keep + 1);
            if (numBytesToTransfer < 0) {
                numBytesToTransfer = 0;
            }

            int mask;
            for(mask = 8; mask <= 8 * numBytesToTransfer; mask += 8) {
                result[i] |= (a[b--] & 255) << mask;
            }

            mask = -1 >>> 8 * (3 - numBytesToTransfer);
            result[i] = ~result[i] & mask;
        }

        for(i = result.length - 1; i >= 0; --i) {
            result[i] = (int)(((long)result[i] & 4294967295L) + 1L);
            if (result[i] != 0) {
                break;
            }
        }

        return result;
    }

    private static int[] makePositive(int[] a) {
        int keep;
        for(keep = 0; keep < a.length && a[keep] == -1; ++keep) {
        }

        int j;
        for(j = keep; j < a.length && a[j] == 0; ++j) {
        }

        int extraInt = j == a.length ? 1 : 0;
        int[] result = new int[a.length - keep + extraInt];

        int i;
        for(i = keep; i < a.length; ++i) {
            result[i - keep + extraInt] = ~a[i];
        }

        for(i = result.length - 1; ++result[i] == 0; --i) {
        }

        return result;
    }

    private int intLength() {
        return (this.bitLength() >>> 5) + 1;
    }

    private int signBit() {
        return this.signum < 0 ? 1 : 0;
    }

    private int signInt() {
        return this.signum < 0 ? -1 : 0;
    }

    private int getInt(int n) {
        if (n < 0) {
            return 0;
        } else if (n >= this.mag.length) {
            return this.signInt();
        } else {
            int magInt = this.mag[this.mag.length - n - 1];
            return this.signum >= 0 ? magInt : (n <= this.firstNonzeroIntNum() ? -magInt : ~magInt);
        }
    }

    private int firstNonzeroIntNum() {
        int fn = this.firstNonzeroIntNumPlusTwo - 2;
        if (fn == -2) {
            int mlen = this.mag.length;

            int i;
            for(i = mlen - 1; i >= 0 && this.mag[i] == 0; --i) {
            }

            fn = mlen - i - 1;
            this.firstNonzeroIntNumPlusTwo = fn + 2;
        }

        return fn;
    }

    private void readObject(ObjectInputStream s) throws IOException, ClassNotFoundException {
        GetField fields = s.readFields();
        int sign = fields.get("signum", -2);
        byte[] magnitude = (byte[])fields.get("magnitude", (Object)null);
        if (sign >= -1 && sign <= 1) {
            int[] mag = stripLeadingZeroBytes(magnitude, 0, magnitude.length);
            if (mag.length == 0 != (sign == 0)) {
                String message = "BigInteger: signum-magnitude mismatch";
                if (fields.defaulted("magnitude")) {
                    message = "BigInteger: Magnitude not present in stream";
                }

                throw new StreamCorruptedException(message);
            } else {
                BigInteger.UnsafeHolder.putSign(this, sign);
                BigInteger.UnsafeHolder.putMag(this, mag);
                if (mag.length >= 67108864) {
                    try {
                        this.checkRange();
                    } catch (ArithmeticException var7) {
                        throw new StreamCorruptedException("BigInteger: Out of the supported range");
                    }
                }

            }
        } else {
            String message = "BigInteger: Invalid signum value";
            if (fields.defaulted("signum")) {
                message = "BigInteger: Signum not present in stream";
            }

            throw new StreamCorruptedException(message);
        }
    }

    private void writeObject(ObjectOutputStream s) throws IOException {
        PutField fields = s.putFields();
        fields.put("signum", this.signum);
        fields.put("magnitude", this.magSerializedForm());
        fields.put("bitCount", -1);
        fields.put("bitLength", -1);
        fields.put("lowestSetBit", -2);
        fields.put("firstNonzeroByteNum", -2);
        s.writeFields();
    }

    private byte[] magSerializedForm() {
        int len = this.mag.length;
        int bitLen = len == 0 ? 0 : (len - 1 << 5) + bitLengthForInt(this.mag[0]);
        int byteLen = bitLen + 7 >>> 3;
        byte[] result = new byte[byteLen];
        int i = byteLen - 1;
        int bytesCopied = 4;
        int intIndex = len - 1;

        for(int nextInt = 0; i >= 0; --i) {
            if (bytesCopied == 4) {
                nextInt = this.mag[intIndex--];
                bytesCopied = 1;
            } else {
                nextInt >>>= 8;
                ++bytesCopied;
            }

            result[i] = (byte)nextInt;
        }

        return result;
    }

    public long longValueExact() {
        if (this.mag.length <= 2 && this.bitLength() <= 63) {
            return this.longValue();
        } else {
            throw new ArithmeticException("BigInteger out of long range");
        }
    }

    public int intValueExact() {
        if (this.mag.length <= 1 && this.bitLength() <= 31) {
            return this.intValue();
        } else {
            throw new ArithmeticException("BigInteger out of int range");
        }
    }

    public short shortValueExact() {
        if (this.mag.length <= 1 && this.bitLength() <= 31) {
            int value = this.intValue();
            if (value >= -32768 && value <= 32767) {
                return this.shortValue();
            }
        }

        throw new ArithmeticException("BigInteger out of short range");
    }

    public byte byteValueExact() {
        if (this.mag.length <= 1 && this.bitLength() <= 31) {
            int value = this.intValue();
            if (value >= -128 && value <= 127) {
                return this.byteValue();
            }
        }

        throw new ArithmeticException("BigInteger out of byte range");
    }

    static {
        int i;
        for(i = 1; i <= 16; ++i) {
            int[] magnitude = new int[]{i};
            posConst[i] = new BigInteger(magnitude, 1);
            negConst[i] = new BigInteger(magnitude, -1);
        }

        powerCache = new BigInteger[37][];
        logCache = new double[37];

        for(i = 2; i <= 36; ++i) {
            powerCache[i] = new BigInteger[]{valueOf((long)i)};
            logCache[i] = Math.log((double)i);
        }

        ZERO = new BigInteger(new int[0], 0);
        ONE = valueOf(1L);
        TWO = valueOf(2L);
        NEGATIVE_ONE = valueOf(-1L);
        TEN = valueOf(10L);
        bnExpModThreshTable = new int[]{7, 25, 81, 241, 673, 1793, 2147483647};
        zeros = new String[64];
        zeros[63] = "000000000000000000000000000000000000000000000000000000000000000";

        for(i = 0; i < 63; ++i) {
            zeros[i] = zeros[63].substring(0, i);
        }

        digitsPerLong = new int[]{0, 0, 62, 39, 31, 27, 24, 22, 20, 19, 18, 18, 17, 17, 16, 16, 15, 15, 15, 14, 14, 14, 14, 13, 13, 13, 13, 13, 13, 12, 12, 12, 12, 12, 12, 12, 12};
        longRadix = new BigInteger[]{null, null, valueOf(4611686018427387904L), valueOf(4052555153018976267L), valueOf(4611686018427387904L), valueOf(7450580596923828125L), valueOf(4738381338321616896L), valueOf(3909821048582988049L), valueOf(1152921504606846976L), valueOf(1350851717672992089L), valueOf(1000000000000000000L), valueOf(5559917313492231481L), valueOf(2218611106740436992L), valueOf(8650415919381337933L), valueOf(2177953337809371136L), valueOf(6568408355712890625L), valueOf(1152921504606846976L), valueOf(2862423051509815793L), valueOf(6746640616477458432L), valueOf(799006685782884121L), valueOf(1638400000000000000L), valueOf(3243919932521508681L), valueOf(6221821273427820544L), valueOf(504036361936467383L), valueOf(876488338465357824L), valueOf(1490116119384765625L), valueOf(2481152873203736576L), valueOf(4052555153018976267L), valueOf(6502111422497947648L), valueOf(353814783205469041L), valueOf(531441000000000000L), valueOf(787662783788549761L), valueOf(1152921504606846976L), valueOf(1667889514952984961L), valueOf(2386420683693101056L), valueOf(3379220508056640625L), valueOf(4738381338321616896L)};
        digitsPerInt = new int[]{0, 0, 30, 19, 15, 13, 11, 11, 10, 9, 9, 8, 8, 8, 8, 7, 7, 7, 7, 7, 7, 7, 6, 6, 6, 6, 6, 6, 6, 6, 6, 6, 6, 6, 6, 6, 5};
        intRadix = new int[]{0, 0, 1073741824, 1162261467, 1073741824, 1220703125, 362797056, 1977326743, 1073741824, 387420489, 1000000000, 214358881, 429981696, 815730721, 1475789056, 170859375, 268435456, 410338673, 612220032, 893871739, 1280000000, 1801088541, 113379904, 148035889, 191102976, 244140625, 308915776, 387420489, 481890304, 594823321, 729000000, 887503681, 1073741824, 1291467969, 1544804416, 1838265625, 60466176};
        serialPersistentFields = new ObjectStreamField[]{new ObjectStreamField("signum", Integer.TYPE), new ObjectStreamField("magnitude", byte[].class), new ObjectStreamField("bitCount", Integer.TYPE), new ObjectStreamField("bitLength", Integer.TYPE), new ObjectStreamField("firstNonzeroByteNum", Integer.TYPE), new ObjectStreamField("lowestSetBit", Integer.TYPE)};
    }

    private static class UnsafeHolder {
        private static final Unsafe unsafe = Unsafe.getUnsafe();
        private static final long signumOffset;
        private static final long magOffset;

        private UnsafeHolder() {
        }

        static void putSign(BigInteger bi, int sign) {
            unsafe.putInt(bi, signumOffset, sign);
        }

        static void putMag(BigInteger bi, int[] magnitude) {
            unsafe.putObject(bi, magOffset, magnitude);
        }

        static {
            signumOffset = unsafe.objectFieldOffset(BigInteger.class, "signum");
            magOffset = unsafe.objectFieldOffset(BigInteger.class, "mag");
        }
    }
}

```
