<template>
    <div class="login-container">
      <h1>Login</h1>
      <form @submit.prevent="login">
        <label for="username">Username:</label>
        <input type="text" id="username" v-model="username" required>
        <label for="password">Password:</label>
        <input type="password" id="password" v-model="password" required>
        <button type="submit">Login</button>
      </form>
    </div>
    <div id="loginInstruction">
      <p>
        After Providing Valid Login Credentials , To get To the Main Page Kindly Refresh the Page. <b> <strong>(Don't Forget to Click on the Login Button)</strong></b>
      </p>
    </div>
  
  </template>
  
  <script>
  export default {
    data() {
      return {
        username: '',
        password: ''
      };
    },
    methods: {
      login() {
        // Replace this with your actual authentication logic
        if (this.username === 'admin' && this.password === 'password') {
          localStorage.setItem('isLoggedIn', 'true');
          this.$emit('loginSuccess'); // Emit the event
        } else {
          alert('Invalid credentials');
        }
      }
    }
  };
  </script>
  
  <style scoped>
  #loginInstruction {
    text-align: center;
    margin-top: 20px;
    border: 3px solid #151414;
    padding: 10px;
    background-color: #ed8181;
  }
  
  .login-container {
    max-width: 400px;
    margin: 0 auto;
    padding: 20px;
    border: 1px solid #ccc;
    border-radius: 5px;
    background-color: #f9f9f9;
  }
  
  h1 {
    text-align: center;
    margin-bottom: 20px;
  }
  
  label {
    display: block;
    margin-bottom: 5px;
  }
  
  input {
    width: 100%;
    padding: 10px;
    margin-bottom: 10px;
    border: 1px solid #ccc;
    border-radius: 5px;
  }
  
  button {
    width: 100%;
    padding: 10px;
    background-color: #4CAF50;
    color: white;
    border: none;
    border-radius: 5px;
    cursor: pointer;
  }
  
  button:hover {
    background-color: #45a049;
  }
  
  button:active {
    background-color: #3e8e41;
  }
  
  button:focus {
    outline: none;
  }
  
  button:disabled {
    background-color: #cccccc;
    cursor: not-allowed;
  }
  
  button:disabled:hover {
    background-color: #cccccc;
  }
  
  button:disabled:active {
    background-color: #cccccc;
  }
  
  .error {
    color: red;
    margin-bottom: 10px;
  }
  
  .success {
    color: green;
    margin-bottom: 10px;
  }
  
  .loading {
    color: blue;
    margin-bottom: 10px;
  }
  
  .logout {
    text-align: center;
    margin-top: 20px;
  }
  
  .logout button {
    background-color: #f44336;
  }
  
  .logout button:hover {
    background-color: #d32f2f;
  }
  
  .logout button:active {
    background-color: #b71c1c;
  }
  
  .logout button:focus {
    outline: none;
  }
  
  .logout button:disabled {
    background-color: #cccccc;
    cursor: not-allowed;
  }
  
  .logout button:disabled:hover {
    background-color: #cccccc;
  }
  
  .logout button:disabled:active {
    background-color: #cccccc;
  }
  
  .logout button:disabled:focus {
    outline: none;
  }
  
  .logout button:disabled {
    background-color: #cccccc;
    cursor: not-allowed;
  }
  
  .logout button:disabled:hover {
    background-color: #cccccc;
  }
  
  .logout button:disabled:active {
    background-color: #cccccc;
  }
  
  .logout button:disabled:focus {
    outline: none;
  }
  </style>