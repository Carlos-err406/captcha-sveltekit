<script>
  import Captcha from "../lib/components/Captcha.svelte";
  const handleValidation = ({ detail }) => {
    console.log(detail);
  };
</script>

<Captcha on:captcha-validation={handleValidation} />
