<template functional>
  <b-row no-gutters class="mt-2">
    <b-col class="text-right attr-title-text" sm="4">{{ props.title }}</b-col>
    <b-col class="text-left attr-value-text" offset="1" sm="7">
      {{ props.value }}
    </b-col>
  </b-row>
</template>

<style>
.attr-title-text {
  color: rgb(126, 138, 148);
}

.attr-value-text {
  font-size: 1.08rem;
  font-weight: 500;
}
</style>
