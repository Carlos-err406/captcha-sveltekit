<script lang="ts">
  import { createEventDispatcher, onMount } from "svelte";
  export let inner: HTMLDivElement | undefined = undefined;
  export let captcha_image: HTMLImageElement | undefined = undefined;
  export let input_text: HTMLInputElement | undefined = undefined;
  export let inner_button: HTMLButtonElement | undefined = undefined;
  export let user_input: string = "";
  export let char_amount: number = 5;
  export let canvas_width: number = 345;
  export let canvas_height: number = 96;
  export let background_color: string = "#FFF";
  export let font: string = "bold 30px Ubuntu";
  export let font_color: string = "#777";
  export let lines_color: string = "#777";
  export let lines_amount: number = 20;
  export let captcha_alt: string = "CAPTCHA";
  export let input_placeholder: string = "CAPTCHA";
  export let button_text: string = "Check";
  export let case_sensitive_captcha: boolean = true;
  export let use_random_font_colors: boolean = false;
  export let use_random_line_colors: boolean = false;
  export let autocheck_user_input: boolean = false;
  export let auto_disapear: boolean = true;
  export let captcha: string = "";
  let ok: boolean = true;
  const dispatch = createEventDispatcher();

  onMount(() => {
    ok = false;
    generateCaptcha();
    createCaptchaImage();
  });

  const generateCaptcha = () => {
    let chars =
      "0123456789abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ";
    for (let i = 0; i < char_amount; i++) {
      captcha += chars.charAt(Math.floor(Math.random() * chars.length));
    }
  };

  const createCaptchaImage = () => {
    let canvas = document.createElement("canvas");
    let ctx = canvas.getContext("2d");
    canvas.width = canvas_width;
    canvas.height = canvas_height;
    if (ctx && captcha_image) {
      ctx.fillStyle = background_color;
      ctx.fillRect(0, 0, canvas.width, canvas.height);
      ctx.font = font;

      for (let i = 0; i < captcha.length; i++) {
        let character = captcha.charAt(i);
        if (!inner) return "";
        let x = (canvas.width / captcha.length) * i + 20;
        let y = canvas.height / 2;
        let angle = Math.random() - 0.5;
        ctx.save();
        ctx.translate(x, y);
        ctx.rotate(angle);
        ctx.fillStyle = font_color;
        if (use_random_font_colors) {
          ctx.fillStyle =
            "rgb(" +
            Math.round(Math.random() * 255) +
            "," +
            Math.round(Math.random() * 255) +
            "," +
            Math.round(Math.random() * 255) +
            ")";
        }
        ctx.fillText(character, -10, 10);
        ctx.restore();
      }

      for (let i = 1; i <= lines_amount; i++) {
        let x1 = Math.random() * canvas.width;
        let y1 = Math.random() * canvas.height;
        let x2 = Math.random() * canvas.width;
        let y2 = Math.random() * canvas.height;
        let color = lines_color;
        if (use_random_line_colors)
          color =
            "rgb(" +
            Math.round(Math.random() * 255) +
            "," +
            Math.round(Math.random() * 255) +
            "," +
            Math.round(Math.random() * 255) +
            ")";
        ctx.beginPath();
        ctx.moveTo(x1, y1);
        ctx.lineTo(x2, y2);
        ctx.strokeStyle = color;
        ctx.stroke();
      }
      captcha_image.src = canvas.toDataURL();
    }
  };

  export const validateCaptcha = () => {
    if (
      (case_sensitive_captcha && user_input === captcha) ||
      (!case_sensitive_captcha &&
        user_input.toLowerCase() === captcha.toLowerCase())
    ) {
      ok = true;
      dispatch("captcha-validation", ok);
    } else {
      ok = false;
      dispatch("captcha-validation", ok);
      user_input = "";
      captcha = "";
      generateCaptcha();
      createCaptchaImage();
    }
  };
  const autovalidate = () => {
    if (
      (case_sensitive_captcha && user_input === captcha) ||
      (!case_sensitive_captcha &&
        user_input.toLowerCase() === captcha.toLowerCase())
    ) {
      ok = true;
      dispatch("captcha-validation", ok);
    }
  };
  $: user_input = user_input.trim();

  $: if (
    autocheck_user_input &&
    user_input.length === captcha.length &&
    user_input
  ) {
    autovalidate();
  }

  $: if (auto_disapear && ok && inner) inner.style.display = "none";
  else if (!auto_disapear && inner) inner.style.display = "inline-flex";
</script>

<div bind:this={inner} class="captcha-container">
  <img src="" alt={captcha_alt} bind:this={captcha_image} />
  <input
    type="text"
    bind:value={user_input}
    placeholder={input_placeholder}
    bind:this={input_text}
  />
  {#if !autocheck_user_input}
    <button on:click={validateCaptcha} bind:this={inner_button}>
      {button_text}
    </button>
  {/if}
</div>

<style>
  .captcha-container {
    display: inline-flex;
    flex-direction: column;
    gap: 15px;
  }

  input {
    border: none;
    outline: 1px solid gray;
    border-radius: 5px;
    padding: 5px;
  }
  input:focus,
  input:active {
    border: none;
    outline: 1px solid gray;
  }
  button {
    padding: 3px;
  }
</style>
