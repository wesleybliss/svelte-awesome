<Svg label={label} width={width} height={height} box={box} style={style}
  spin={spin} flip={flip} inverse={inverse} pulse={pulse}>
  <slot>
    {#if self}
      {#if self.paths}
        {#each self.paths as path, i}
        <Path id="{i}" data="{path}"/>
        {/each}
      {/if}
      {#if self.polygons}
        {#each self.polygons as polygon, i}
        <Polygon id="{i}" data="{polygon}"/>
        {/each}
      {/if}
      {#if self.raw}
        <Raw bind:data={self} />
      {/if}
    {/if}
  </slot>
</Svg>

<script>
  /* eslint-disable no-unused-vars */

  import { afterUpdate } from 'svelte';

  import Path from './svg/Path.svelte';
  import Polygon from './svg/Polygon.svelte';
  import Raw from './svg/Raw.svelte';
  import Svg from './svg/Svg.svelte';

  export let data;
  export let scale = 1;
  export let spin = false;
  export let inverse = false;
  export let pulse = false;
  export let flip = null;
  export let label = null;
  export let self = null;

  // internal
  let x = 0;
  let y = 0;
  let childrenHeight = 0;
  let childrenWidth = 0;
  let outerScale = 1;

  let width;
  let height;
  let style;
  let box;

  function init() {
    if (typeof data === 'undefined') {
      return;
    }
    const [name] = Object.keys(data);
    const icon = data[name];
    if (!icon.paths) {
      icon.paths = [];
    }
    if (icon.d) {
      icon.paths.push({
        d: icon.d,
      });
    }
    if (!icon.polygons) {
      icon.polygons = [];
    }
    if (icon.points) {
      icon.polygons.push({
        points: icon.points,
      });
    }
    self = icon;
  }

  function normalisedScale() {
    let numScale = 1;
    if (typeof scale !== 'undefined') {
      numScale = Number(scale);
    }
    if (isNaN(numScale) || numScale <= 0) { // eslint-disable-line no-restricted-globals
      console.warn('Invalid prop: prop "scale" should be a number over 0.'); // eslint-disable-line no-console
      return outerScale;
    }
    return numScale * outerScale;
  }
  function calculateBox() {
    if (self) {
      return `0 0 ${self.width} ${self.height}`;
    }
    return `0 0 ${width} ${height}`;
  }
  function calculateRatio() {
    if (!self) {
      return 1;
    }
    return Math.max(self.width, self.height) / 16;
  }
  function calculateWidth() {
    if (childrenWidth) {
      return childrenWidth;
    }
    if (self) {
      return (self.width / calculateRatio()) * normalisedScale();
    }
    return 0;
  }
  function calculateHeight() {
    if (childrenHeight) {
      return childrenHeight;
    }
    if (self) {
      return (self.height / calculateRatio()) * normalisedScale();
    }
    return 0;
  }
  function calculateStyle() {
    let size = normalisedScale();
    if (size === 1) {
      return false;
    }
    return `font-size: ${size}em`;
  }

  afterUpdate(() => {
    init();
    width = calculateWidth();
    height = calculateHeight();
    style = calculateStyle();
    box = calculateBox();
  });
</script>
