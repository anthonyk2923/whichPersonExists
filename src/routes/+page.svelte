<script>
  import realData from '$lib/realData.js';
  import { onMount, afterUpdate, onDestroy } from 'svelte';

  let imgOrder;
  let correctAnswersCount = 0;
  let imgSrc = '';
  let imgSrc2 = '';

  let isImg1Shrunk = false;
  let isImg2Shrunk = false;

  let img1, img2;
  let rect1, rect2;
  let loading = true;
  let isCheat = 'imgZero'

  async function findRealImg() {
    const imgData = await realData[Math.floor(Math.random() * 500)];
    const fileId = new URL(imgData.image.file_url).searchParams.get('id');
    const img = `https://drive.google.com/thumbnail?id=${fileId}&sz=w1000-h1000`
    return img;
  }

  function updateImageRects() {
    if (img1 && img2) {
      rect1 = img1.getBoundingClientRect();
      rect2 = img2.getBoundingClientRect();
    }
  }

  async function createOrder() {
    imgOrder = Math.floor(Math.random() * 2);
    if (imgOrder === 0) {
      imgSrc = '';
      imgSrc2 = '';
      imgSrc = await findRealImg();
      imgSrc2 =`https://thispersondoesnotexist.com?timestamp=${Date.now()}`
    } else {
      imgSrc = '';
      imgSrc2 = '';
      imgSrc = `https://thispersondoesnotexist.com?timestamp=${Date.now()}`
      imgSrc2 = await findRealImg();
    }
    loading = true;  // Start loading state
  }

  function checkIfImagesLoaded() {
    if (img1.complete && img2.complete) {
      loading = false;  // Stop loading state
      updateImageRects();
    }
  }

  onMount(async () => {
    await createOrder();

    if (typeof window !== 'undefined') {
      window.addEventListener('resize', updateImageRects);
    }
  });

  afterUpdate(() => {
    updateImageRects();
  });

  onDestroy(() => {
    if (typeof window !== 'undefined') {
      window.removeEventListener('resize', updateImageRects);
    }
  });

  function checkImg(event, mo1, mo2) {
    mo1 = rect1.left <= event.clientX && event.clientX <= rect1.right &&
          rect1.top <= event.clientY && event.clientY <= rect1.bottom;
    mo2 = rect2.left <= event.clientX && event.clientX <= rect2.right &&
          rect2.top <= event.clientY && event.clientY <= rect2.bottom;
    return [mo1, mo2];
  }

  let isMouseOverImg1;
  let isMouseOverImg2;

  function handleMouseMove(event) {
    if (!rect1 || !rect2) return;

    [isMouseOverImg1, isMouseOverImg2] = checkImg(event, isMouseOverImg1, isMouseOverImg2);

    if (isMouseOverImg1) {
      if (!isImg1Shrunk) {
        img1.style.transform = 'scale(0.9)';
        isImg1Shrunk = true;
      }
    } else {
      if (isImg1Shrunk) {
        img1.style.transform = 'scale(1)';
        isImg1Shrunk = false;
      }
    }

    if (isMouseOverImg2) {
      if (!isImg2Shrunk) {
        img2.style.transform = 'scale(0.9)';
        isImg2Shrunk = true;
      }
    } else {
      if (isImg2Shrunk) {
        img2.style.transform = 'scale(1)';
        isImg2Shrunk = false;
      }
    }
  }

  function handleClick() {
    if (isMouseOverImg1) {
      submitAnswer(0);
    } else if (isMouseOverImg2) {
      submitAnswer(1);
    } else {
      return;
    }
  }

  function submitAnswer(answer) {
    answer === imgOrder ? correctAnswer() : incorrectAnswer();
  }

  function correctAnswer() {
    imgOrder = 2;
    correctAnswersCount += 1;
    createOrder();
  }

  function incorrectAnswer() {
    imgOrder !== 2 && (correctAnswersCount = 0);
    imgOrder = 2;
    createOrder();
  }
</script>

<button class="relative w-full h-screen flex flex-col md:flex-row bg-zinc-950 " on:mousemove={handleMouseMove} on:click={handleClick}>
  <div class="w-full md:w-1/2 h-1/2 md:h-full" style:display={loading ? 'none' : 'block'}>
    <img  bind:this={img1} src={imgSrc} alt="img1" class="{isCheat} object-cover w-full h-full transition-transform duration-300 rounded-md" on:load={checkIfImagesLoaded}> 
  </div>
  <div class="w-full md:w-1/2 h-1/2 md:h-full" style:display={loading ? 'none' : 'block'}>
    <img bind:this={img2} src={imgSrc2} alt="img2" class="{isCheat} object-cover w-full h-full  transition-transform duration-300 rounded-md " on:load={checkIfImagesLoaded}>
  </div>
  {#if loading}
    <div class="absolute inset-0 flex flex-col items-center justify-center text-white bg-black bg-opacity-40  ">
      <h1 class="text-4xl font-bold">Loading...</h1>
    </div>
  {:else}
    <div class="absolute inset-0 flex flex-col items-center justify-center text-white bg-black bg-opacity-40">
      <h1 class="absolute inset-14 text-3xl font-bold text-white">{correctAnswersCount}</h1>
      <h1 class="text-4xl font-bold">Which Person Exists?</h1>
      <h1 class="text-2xl">Select The Real Person</h1>
    </div>
  {/if}
</button>
<style>
  .imgZero {
  outline: 2vh solid black;
  outline-offset: -2vh;
  border-radius: 30px;
}

</style>