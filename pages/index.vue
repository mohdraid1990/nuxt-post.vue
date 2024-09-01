<template>
  <div class="main">
    <h1>Welcome to Nuxt 3!</h1>
    <nuxt-link to="/posts" class="button">Go to Posts</nuxt-link>
  </div>
</template>

<style scoped>


.main {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  background-color: rgba(0, 0, 0, 0.5);
  padding: 30px;
  border-radius: 10px;
  box-shadow: 0 4px 10px rgba(0, 0, 0, 0.2);
  height: auto;
}

h1 {
  margin: 0 0 20px;
  font-size: 2.5rem;
}

.button {
  background-color: #ff6f61;
  color: white;
  padding: 12px 20px;
  border-radius: 5px;
  text-decoration: none;
  font-size: 1rem;
  transition: background-color 0.3s;
}

.button:hover {
  background-color: #ff4a3d;
}
</style>
