<script setup>
import { toRaw, onMounted, ref } from "vue";

const carouselContainer = ref(null);
const carouselItems = ref([]);
const counter = ref(null);
let index = 1;
let interval = null;
let dragStartX = null;
let snapTo = null;
let lastCursorX = null;

const mapData = (data, imageProp, altProp) => {
  if (data.data.value === null) return [];
  return toRaw(data.data.value).map((object) => ({
    image: object[imageProp],
    alt: object[altProp],
  }));
};

const setCarouselTransform = (transition) => {
  if (carouselContainer.value) {
    carouselContainer.value.style.transition = transition;
    carouselContainer.value.style.transform = `translateX(-${index * 100}%)`;
  }
};

const shiftSlide = (toSide = null, toIndex = 1) => {
  if (toSide !== null) toSide === "left" ? (index -= 1) : (index += 1);
  else index = toIndex;

  setCarouselTransform("transform 0.3s ease");

  if (index === 0) {
    index = carouselItems.value.length - 2;
    setTimeout(() => {
      setCarouselTransform("none");
    }, 300);
  }
  if (index >= carouselItems.value.length - 1) {
    index = 1;
    setTimeout(() => {
      setCarouselTransform("none");
    }, 300);
  }
  if (counter.value) counter.value.innerHTML = index < 10 ? "0" + index : index;
  if (interval) clearInterval(interval);
  interval = setInterval(goRight, 5000);
  useCookie("index").value = index;
};

const goLeft = () => {
  shiftSlide("left");
};

const goRight = () => {
  shiftSlide("right");
};

const drag = (x) => {
  const containerWidth = carouselContainer.value.clientWidth;
  const transform = carouselContainer.value.style.transform;
  const unit = transform.includes("%") ? "%" : "px";
  const translateX = transform.substring(
    12,
    transform.length - 1 - unit.length
  );

  const currentContainerPosPx =
    unit === "%" ? (translateX / 100) * containerWidth : translateX;

  const currentContainerPosProc =
    unit === "px" ? (translateX / containerWidth) * 100 : translateX;

  snapTo = Math.round(currentContainerPosProc / 100);
  const cursorDiffX = lastCursorX !== null ? -1 * (lastCursorX - x) : 0;
  lastCursorX = x;
  let moveTo = currentContainerPosPx - cursorDiffX;

  if (dragStartX === null) dragStartX = currentContainerPosPx;
  else {
    if (moveTo > dragStartX + containerWidth)
      moveTo = dragStartX + containerWidth;
    else if (moveTo < dragStartX - containerWidth)
      moveTo = dragStartX - containerWidth;
  }

  carouselContainer.value.style.transition = "none";
  carouselContainer.value.style.transform = `translateX(-${moveTo}px)`;

  if (interval) clearInterval(interval);
};

const mouseUp = () => {
  dragStartX = null;
  lastCursorX = null;
  if (snapTo !== null) {
    shiftSlide(null, snapTo);
    snapTo = null;
  }
};

const mouseMove = (event) => {
  if (event.buttons === 1) drag(event.clientX);
};

const touchMove = (event) => {
  drag(event.touches[0].clientX);
};

const mountains = await useFetch("https://api.nuxtjs.dev/mountains");
const mappedMountains = mapData(mountains, "image", "title");

const rivers = await useFetch("https://api.nuxtjs.dev/rivers");
const mappedRivers = mapData(rivers, "image", "title");

const planets = await useFetch("https://api.nuxtjs.dev/planets");
const mappedPlanets = mapData(planets, "image", "title");

const slides = mappedMountains.concat(mappedRivers, mappedPlanets).slice(0, 20);
slides.unshift(slides[slides.length - 1]);
slides.push(slides[1]);

const cookieValue = useCookie("index").value;
if (cookieValue) index = cookieValue;

onMounted(() => {
  interval = setInterval(goRight, 5000);
});
</script>

<template>
  <div class="background">
    <div class="carousel">
      <div
        ref="carouselContainer"
        class="carousel__container"
        :style="
          'transition: none 0s ease 0s; transform: translateX(-' +
          index * 100 +
          '%);'
        "
        @mousemove="mouseMove"
        @mouseup="mouseUp"
        @touchmove="touchMove"
        @touchend="mouseUp"
      >
        <div
          ref="carouselItems"
          class="carousel__item"
          v-for="(slide, i) in slides"
          v-bind:key="i"
        >
          <img :src="slide.image" :alt="slide.alt" draggable="false" />
        </div>
      </div>
      <div class="carousel__navigation">
        <div class="arrows">
          <span class="arrow arrow-left" @click="goLeft">
            <svg>
              <use xlink:href="./assets/img/sprite.svg#icon-arrow-left"></use>
            </svg>
          </span>
          <span class="arrow arrow-right" @click="goRight">
            <svg>
              <use xlink:href="./assets/img/sprite.svg#icon-arrow-right"></use>
            </svg>
          </span>
        </div>
        <div class="counter">
          <span class="counter__current-slide" ref="counter">{{
            index < 10 ? "0" + index : index
          }}</span>
          <span class="counter__slash">/</span>
          <span class="counter__max-slide">{{ slides.length - 2 }}</span>
        </div>
      </div>
    </div>
  </div>
</template>

<style lang="scss">
.background {
  display: flex;
  justify-content: center;
  align-items: center;
  height: 100vh;
  width: 100vw;
  display: flex;
  .carousel {
    display: flex;
    justify-content: center;
    align-items: center;
    flex-direction: column;
    max-width: 1000px;
    overflow: hidden;
    margin: 0 20px;

    &__container {
      display: flex;
      max-height: 100vh;
      height: 500px;
    }

    &__item {
      position: relative;
      width: 100%;
      flex: 1 0 100%;
      cursor: grabbing;
      img {
        width: 100%;
        height: 100%;
        object-fit: cover;
        object-position: center;
        border-radius: 30px;
        box-shadow: 2px 2px 10px rgba(0, 0, 0, 0.2);
      }
    }

    &__navigation {
      max-width: 1000px;
      width: 100%;
      display: flex;
      justify-content: space-between;
      align-items: center;
      padding: 20px;
      .arrows {
        .arrow {
          cursor: pointer;
          svg {
            width: 25px;
            height: 25px;
            fill: #fff;
            stroke: var(--color-primary);
          }
        }
      }
      .counter {
        &__current-slide {
          font-family: "Stolzl-Bold", sans-serif;
          color: var(--color-primary);
        }
        &__slash {
          margin: 0 3px;
        }
      }
    }
  }
}
</style>
