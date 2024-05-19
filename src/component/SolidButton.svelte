<script>
  let buttonProps = {
    class: [($$restProps.class || "") + " solid-button"]
  };
  export let style = "";
  export let disabled = false;
</script>

<button
  {style}
  {disabled}
  class=""
  on:click
  on:mouseover
  on:mouseenter
  on:mouseleave
  {...buttonProps}
>
  <slot />
</button>

<style>
  .solid-button {
    border: none;
    border-radius: 5px;
    text-align: center;
    cursor: pointer;
    background: #666666;
    color: #ffffff;
  }

  .sm {
    font-size: 12px;
    line-height: 12px;
    padding: 8px 16px;
    border-radius: 5px;
  }

  .xs {
    font-size: 12px;
    line-height: 12px;
    padding: 6px 10px;
    border-radius: 3px;
  }

  .round {
    border-radius: 9999px;
  }
</style>
