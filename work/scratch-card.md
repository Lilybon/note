# Scratch Card

## What is it?

Excerpt from [Cambridge Dictionary](https://dictionary.cambridge.org/dictionary/english/scratch-card), a scratch card is as defined below.

![scratch card](https://dictionary.cambridge.org/images/full/scratc_noun_002_32489.jpg?version=5.0.305 =300x)

> a small card that you can buy to try to win a prize, with a thin layer that you rub off in order to see if you have winning numbers on it.

## Feature

A scratch card which has 3 \* 3 reward image list, the final reward for user will appears 3 times, as for other reward, they will appears 2 times or below.

| Un-scratch                                                          | Scratched                                                           |
| ------------------------------------------------------------------- | ------------------------------------------------------------------- |
| ![customized scratch card 1](https://i.imgur.com/OfDTaV8.png =300x) | ![customized scratch card 2](https://i.imgur.com/KuE9vMS.png =300x) |

## Survey

### How to create a scratch card component step by step?

1. Binding event for different device (ducktype is fine):
   Mobile: `touchstart`, `touchmove`, `touchend`
   Desktop: `mousedown`, `mousemove`, `mouseup`
2. Load brush image and remember to set globalCompositeOperation as `destination-out` while drawing brush image on canvas.
3. Calculate transparent (alpha value) pixels in 2d context / all pixels in 2d context to define whether to show up all card.

### Library (Source Code)

Here are some nice libraries to learn how to implement scratch card feature.

- [ScratchCard](https://github.com/Masth0/ScratchCard)
- [lucky-card](https://github.com/Franslee/lucky-card)
- [scrapAward-dev.js](https://github.com/zxpsuper/Demo/blob/master/letter/scrapAward-dev.js)
- [vue-scratch-card](https://github.com/ZENGzoe/vue-scratch-card/blob/master/src/packages/scratch-card/scratch-card.vue)

### Conclusion

It's fine to use library if a scratch card contains single image to calculate show-up mechanism. However, if it contains a 3 \* 3 grid images to calculate show-up mechanism (e.g. same image shows up 3 times), then I'd better customize it by myself.

## Implmentation

I won't explain all the details to build a scratch card since there are already multiple articles and libraries to teach how to build a simple scratch card step by step.

Let's focus on the tricky part of the 3 \* 3 grid images scratch card.

### Template

Scratch wrapper will be covered by foreground canvas, and background template can be customized.

```html
<div ref="scratchWrapper" class="scratch-card">
  <div class="scratch-card__background">
    <slot />
  </div>
  <canvas ref="scratchCanvas" class="scratch-card__foreground" />
</div>
```

### Use Promise to Load Image

If multiple image sources are required, it's good to use a promise to make sure an image before canvas initialization.

```typescript
const loadImage = (src: string): Promise<HTMLImageElement> =>
  new Promise((resolve) => {
    const image = new Image()
    image.onload = () => {
      resolve(image)
    }
    image.src = src
  })
```

```typescript
import { defineComponent } from 'vue'

import scratch from '~/assets/img/scratch-card/scratch.png'
import brush from '~/assets/img/scratch-card/brush.png'

export default defineComponent({
  async setup() {
    const [scratchImage, brushImage] = await Promise.all([
      loadImage(scratch),
      loadImage(brush),
    ])

    // do some canvas initialization
    // ...
  },
})
```

### Calculate Area

How to calculate whether each grid is opened

1. Use a throttle to prevent too much calculation on move event.
2. Get 2d context imageData.
3. Get 2d context alpha data.
4. Identify grid all pixels and transparent pixels.
5. Calculate open ratio of each grid.
6. Define whether each grid is opened or not.

p.s. For the grid index calculation, it's almost the same with leetcode problem [36. Valid Sudoku](https://leetcode.com/problems/valid-sudoku/) but the grid is larger.

```typescript
import { defineComponent, ref } from 'vue'
import type { Ref } from 'vue'
import type { Item } from '@/types/scratch-card'

export default defineComponent({
    const THROTTLE_MILLISECONDS = 400

    const items: Ref<Array<Item>> = ref(Array.from(Array(9), (_, index) => {
        cost id = index < 3 ? -1 : index
        return {
            image: `IMAGE_URL_${index % 3}`,
            title: 'IMAGE_TITLE',
            id
        }
    })

    // Scratch card foreground canvas ref
    const scratchCanvas: Ref<HTMLCanvasElement> = ref(null)

    // Step 1
    const calculateArea = throttle((): void => {
        // Step 2
        const context = scratchCanvas.value.getContext('2d')!
        const imageData: Ref<> = context.getImageData(0, 0, scratchCanvas.width, scratchCanvas.height)

        // Data is stored as a one-dimensional array in the RGBA order, with integer values between 0 and 255 (inclusive)
        const RGBA_LENGTH = 4

        // Alpha treshold to judege whether an alpha value is transparent or not
        const TRANSPARENT_ALPHA_TRESHOLD = 75

        // Ratio threshold to judge whether a grid is opened
        const OPEN_RATIO_TRESHOLD = 0.5

        const SUB_EDGE_PER_EDGE = 3

        const INDEX_OF_TRANSPARENT_PIXELS = 0

        const INDEX_OF_ALL_PIXELS = 1

        // Step 3
        const alphas = imageData.data.filter((_item, index) => (index + 1) % RGBA_LENGTH === 0)

        // Since image is square, use sqrt to get edge width
        const pixelsPerEdge = Math.sqrt(alphas.length)

        // Since grid is 3 * 3, divide 3 to get sub edge width
        const pixelsPerSubEdge = pixelsPerEdge / SUB_EDGE_PER_EDGE

        const isRewardOpen = alphas
            // Step 4
            .reduce(
                (counts, alpha, alphaIndex) => {
                    // Transform alpha index to [x, y] in pixel grid
                    const remainder = alphaIndex % pixelsPerEdge
                    const quotient = (alphaIndex - remainder) / pixelsPerEdge

                    // Transform [x, y] in pixel grid to grid index
                    const countIndex =
                        Math.floor(quotient / pixelsPerSubEdge) * SUB_EDGE_PER_EDGE +
                        Math.floor(remainder / pixelsPerSubEdge)

                    counts[INDEX_OF_ALL_PIXELS]++
                    if (alpha <= TRANSPARENT_ALPHA_TRESHOLD) {
                      counts[INDEX_OF_TRANSPARENT_PIXELS]++
                    }

                    return counts
                },
                Array.from(Array(items.value.length), () => ([0, 0]))
            )
            // Step 5
            .map(([transparentPixels, allPixels]) => transparentPixels / allPixels)
            // Step 6
            .map((ratio) => ratio >= OPEN_RATIO_TRESHOLD)

        // use isRewardOpen to calculate max times for same image occurs
        // ...
    }, THROTTLE_MILLISECONDS)


})
```

### Keep Scratch Trace After Resizing

How to keep scratch trace:

1. If canvas width is equal to wrapper width and canvas height is equal to wrapper height, don't do anything.
2. Create a data url and load image based on old canvas
3. (Resize canvas and paint initial canvas)
4. Append trace on resized canvas

```typescript
import { defineComponent, ref } from 'vue'
import type { Ref } from 'vue'

export default defineComponent({
    const scratchWrapper: Ref<HTMLDivElement> = ref(null)
    const scratchCanvas: Ref<HTMLCanvasElement> = ref(null)

    const onResize = async (_event: UIEvent): Promise<void> => {
        // Step 1
        if (
            scratchCanvas.value.width === scratchWrapper.value.clientWidth &&
            scratchCanvas.value.height === scratchWrapper.value.clientHeight
        ) {
            return
        }

        // Step 2
        const context = scratchCanvas.value.getContext('2d')!
        const dataUrl = scratchCanvas.value.toDataURL('image/png')
        const image = await loadImage(dataUrl)

        // Step 3
        // ...

        context.save()

        // Step 4
        context.globalCompositeOperation = 'destination-in'
        context.drawImage(image, 0, 0, scratchCanvas.value.width, scratchCanvas.value.height)

        context.restore()
    }
})
```

## Reference

- [实现一个“刮刮乐”游戏](https://zxpsuper.github.io/advanced_front_end/book/canvas/canvas4.html#_1-%E8%83%8C%E6%99%AF)
- [理解 Canvas Context 的 save() 和 restore()](https://juejin.cn/post/6844903879599996942)
- [Compositing and clipping](https://developer.mozilla.org/en-US/docs/Web/API/Canvas_API/Tutorial/Compositing)
- [CanvasRenderingContext2D.getImageData()](https://developer.mozilla.org/en-US/docs/Web/API/CanvasRenderingContext2D/getImageData)
- [CanvasRenderingContext2D.createPattern()](https://developer.mozilla.org/en-US/docs/Web/API/CanvasRenderingContext2D/createPattern)
- [HTMLCanvasElement.toDataURL()](https://developer.mozilla.org/zh-TW/docs/Web/API/HTMLCanvasElement/toDataURL)

###### tags: `Work` `Scratch Card` `Vue3`
