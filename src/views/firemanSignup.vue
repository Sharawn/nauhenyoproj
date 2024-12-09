<template>
  <div class="signup-container">
    <!-- Logo Image at the top center -->
    <div class="logo-container">
      <img src="@/assets/naulogo.png" alt="Logo" class="logo">
    </div>

    <!-- Form with the title inside -->
    <form @submit.prevent="handleSubmit" class="signup-form">
      <h1>Sign Up</h1>
      <div class="form-group">
        <label for="fullname">Full Name</label>
        <input id="fullname" v-model="form.fullname" type="text" required>
      </div>

      <div class="form-group">
        <label for="birthday">Birthday</label>
        <input id="birthday" v-model="form.birthday" type="date" required>
      </div>

      <div class="form-row">
        <div class="form-group half-width">
          <label for="gender">Gender</label>
          <select id="gender" v-model="form.gender" required>
            <option value="">Select Gender</option>
            <option value="male">Male</option>
            <option value="female">Female</option>
            <option value="other">Other</option>
          </select>
        </div>

        <div class="form-group half-width">
          <label for="contactNumber">Contact Number</label>
          <input id="contactNumber" v-model="form.contactNumber" type="tel" required>
        </div>
      </div>

      <div class="form-group">
        <label for="email">Email</label>
        <input id="email" v-model="form.email" type="email" required>
      </div>

      <div class="form-group">
        <label for="password">Password</label>
        <div class="password-input">
          <input 
            id="password" 
            v-model="form.password" 
            :type="showPassword ? 'text' : 'password'" 
            required
          >
          <button type="button" @click="showPassword = !showPassword" class="toggle-password">
            <EyeIcon v-if="!showPassword" />
            <EyeOffIcon v-else />
          </button>
        </div>
      </div>

      <button type="submit" class="submit-button">Sign Up</button>

      <div v-if="error" class="error-message">
        {{ error }}
      </div>
    </form>

    <!-- Sign-up Progress Modal -->
    <div v-if="isSigningUp" class="fixed inset-0 bg-black bg-opacity-50 flex items-center justify-center z-50">
      <div class="bg-white p-8 rounded-lg shadow-xl max-w-md w-full">
        <h2 class="text-2xl font-bold mb-4 text-center">Signing Up</h2>
        <div class="mb-4">
          <div class="h-2 bg-gray-200 rounded-full">
            <div
              class="h-full bg-blue-600 rounded-full transition-all duration-500 ease-out"
              :style="{ width: `${signupProgress}%` }"
            ></div>
          </div>
        </div>
        <p class="text-center text-gray-600">{{ signupStatus }}</p>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref } from 'vue';
import { useRouter } from 'vue-router';
import { EyeIcon, EyeOffIcon } from 'lucide-vue-next';
import { auth, db } from '@/firebase/config';
import { createUserWithEmailAndPassword } from 'firebase/auth';
import { doc, setDoc } from 'firebase/firestore';

const router = useRouter();
const form = ref({
  fullname: '',
  birthday: '',
  gender: '',
  contactNumber: '',
  email: '',
  password: ''
});

const showPassword = ref(false);
const error = ref('');
const isSigningUp = ref(false);
const signupProgress = ref(0);
const signupStatus = ref('');

const handleSubmit = async () => {
  isSigningUp.value = true;
  signupProgress.value = 0;
  signupStatus.value = 'Creating your account...';
  error.value = '';

  try {
    // Create user with email and password
    signupProgress.value = 25;
    const userCredential = await createUserWithEmailAndPassword(auth, form.value.email, form.value.password);
    const user = userCredential.user;

    signupProgress.value = 50;
    signupStatus.value = 'Account created. Saving your details...';

    // Store additional user details in Firestore's firefighters collection
    await setDoc(doc(db, 'firefighters', user.uid), {
      fullname: form.value.fullname,
      birthday: form.value.birthday,
      gender: form.value.gender,
      contactNumber: form.value.contactNumber,
      email: form.value.email,
      createdAt: new Date().toISOString(),
      uid: user.uid
    });

    signupProgress.value = 100;
    signupStatus.value = 'Sign up successful!';

    console.log('Firefighter signed up successfully');
    
    // Delay to show the completed progress bar
    setTimeout(() => {
      isSigningUp.value = false;
      router.push('/login');
    }, 1000);

  } catch (e) {
    console.error('Error during signup:', e);
    error.value = e.message || 'An error occurred during sign up. Please try again later.';
    isSigningUp.value = false;
    signupProgress.value = 0;
    signupStatus.value = '';
  }
};
</script>

<style scoped>
@import url('https://fonts.googleapis.com/css2?family=Poppins:wght@400;600&display=swap');

* {
  font-family: 'Poppins', sans-serif;
}

.signup-container {
  max-width: 500px;
  margin: 0 auto;
  padding: 20px;
  text-align: center;
}

.logo-container {
  margin-bottom: 20px;
}

.logo {
  width: 100px;
}

.signup-form {
  background-color: rgba(249, 249, 249, 0.9);
  padding: 30px;
  border-radius: 8px;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.2);
  text-align: left;
}

h1 {
  color: #333;
  margin-bottom: 20px;
  text-align: center;
}

.form-group {
  margin-bottom: 15px;
}

.form-row {
  display: flex;
  gap: 15px;
}

.half-width {
  flex: 1;
}

label {
  display: block;
  margin-bottom: 5px;
  color: #555;
  text-align: left;
}

input, select {
  width: 100%;
  padding: 8px;
  border: 1px solid #ddd;
  border-radius: 4px;
  font-size: 16px;
}

.password-input {
  position: relative;
}

.toggle-password {
  position: absolute;
  right: 8px;
  top: 50%;
  transform: translateY(-50%);
  background: none;
  border: none;
  cursor: pointer;
  color: #555;
}

.submit-button {
  width: 100%;
  padding: 10px;
  background-color: #062654;
  color: white;
  border: none;
  border-radius: 4px;
  font-size: 16px;
  cursor: pointer;
  transition: background-color 0.3s;
}

.submit-button:hover {
  background-color: #f58e08;
}

.error-message {
  color: red;
  margin-top: 10px;
  text-align: center;
}
</style>