# Convolution

**Tags:** math, layers, cnn

---

## What it is

Convolution is a mathematical operation used to find relevant features in an input matrix.

It uses a **filter** (also called a **kernel**), which is a small grid of weight values. The filter aligns with a patch of the input that has the same size and computes a **dot product** (multiply the matching numbers, then add them all up). That gives one number. The filter then slides across all the input values, and the numbers it produces together form a **feature map**.

Each filter specializes in one particular feature, and its output at a position tells how strongly that feature is present in that patch of the input.

## What amazed me

The filter is **learned** (through backpropagation), not designed by hand. The network figures out for itself which patterns are worth detecting.

## Colour images and multiple filters

If the input is a colour image, its shape looks like, for example, `224 × 224 × 3` (height × width × channels). The filter also has depth 3 (for example `3 × 3 × 3`), so it covers a patch across all three channels, and it still gives **one value for each position**.

A convolution layer has multiple filters, each looking for a different feature on the same input. Each filter produces its own feature map, and these are stacked together. So the **depth of the output equals the number of filters**.

---

## Stride

**Stride** is how many positions the filter moves each time it slides.

- Stride 1: the filter moves one position at a time.
- Stride 2: the filter skips every other position, so the output becomes smaller.

## Padding

**Padding** means adding a border of extra values (usually zeros) around the input before applying the filter.

- **Valid padding:** no padding (`p = 0`). The output shrinks.
- **Same padding:** enough padding that the output keeps the input's size (for a 3 × 3 filter with stride 1, that is `p = 1`).

Why it is useful: without padding the feature map shrinks with every layer, and the pixels at the edges are used by the filter far less often than pixels in the centre.

## Output size

For a square input of size `n`, a filter of size `k`, padding `p` and stride `s`:

$$
\text{output size} = \left\lfloor \frac{n - k + 2p}{s} \right\rfloor + 1
$$

(The floor `⌊ ⌋` means: round down if the division is not exact.)

For a colour image: input `224 × 224 × 3`, with 64 filters of size `3 × 3 × 3`, padding 1, stride 1:

- Height and width: (224 − 3 + 2)/1 + 1 = 224
- Depth: 64 (one feature map per filter)
- Output: `224 × 224 × 64`

---

## Worked example: 2D image

**Input** (5 × 5): the left side is bright (1) and the right side is dark (0), so there is a vertical edge in the image.

```
1 1 1 0 0
1 1 1 0 0
1 1 1 0 0
1 1 1 0 0
1 1 1 0 0
```

**Filter** (3 × 3), a vertical edge detector:

```
1  0  -1
1  0  -1
1  0  -1
```

**Settings:** stride 1, padding 0, so the output size is (5 − 3)/1 + 1 = 3, giving a 3 × 3 feature map.

**Calculation for the top row of positions.** Every row of the patch is multiplied with `[1, 0, -1]`, and the three rows give the same result, so the total is 3 × (one row's result):

| Position | Each patch row | One row's result | Total |
|----------|----------------|------------------|-------|
| col 0 | `1 1 1` | 1×1 + 1×0 + 1×(−1) = 0 | **0** (no edge) |
| col 1 | `1 1 0` | 1×1 + 1×0 + 0×(−1) = 1 | **3** (edge found) |
| col 2 | `1 0 0` | 1×1 + 0×0 + 0×(−1) = 1 | **3** (edge found) |

The next two rows of positions give the same values, so the **feature map** is:

```
0  3  3
0  3  3
0  3  3
```

The output is 0 where the input is flat and positive where the input changes from bright (left) to dark (right), so this filter works as an edge detector. In a CNN, the filter numbers start random and backpropagation adjusts them until they detect something useful.
