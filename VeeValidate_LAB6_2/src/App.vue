<template>
  <div class="registration-form">
    <h2>Регистрация</h2>

    <form @submit.prevent="onSubmit">
      <!-- Почта/Логин -->
      <div class="field-group">
        <label for="email">Email:</label>
        <input
          id="email"
          type="email"
          v-model="emailValue"
          :class="emailFieldClass"
          placeholder="example@domain.com"
        />
      </div>

      <!-- Пароль -->
      <div class="field-group password-group">
        <label for="password">Пароль:</label>
        <div class="password-wrapper">
          <input
            id="password"
            type="password"
            v-model="passwordValue"
            :class="passwordFieldClass"
            placeholder="Введите пароль"
          />
          <ul class="criteria-list">
            <li
              v-for="crit in passwordCriteria"
              :key="crit.text"
              :class="crit.valid ? 'valid' : 'invalid'"
            >
              {{ crit.text }}
            </li>
          </ul>
        </div>
      </div>

      <!-- Чекбокс(согласие ) -->
      <div class="field-group checkbox-group">
        <label>
          <input type="checkbox" v-model="agreed" />
          I agree with license agreement
        </label>
      </div>

      <!-- Кнопка -->
      <button type="submit" :disabled="!isFormValid">Зарегистрироваться</button>
    </form>
  </div>
</template>

<script setup>
import { ref, computed } from 'vue';


const emailValue = ref('');
const passwordValue = ref('');
const agreed = ref(false);

// проверка почты 
const isEmailValid = computed(() => {
  const email = emailValue.value;
  if (!email) return false;
  const regex = /^[^\s@]+@([^\s@]+\.)+[^\s@]+$/;
  return regex.test(email);
});

// проверка соответствия пароля 
const isPasswordValid = computed(() => {
  const pwd = passwordValue.value;
  if (!pwd) return false;
  return (
    pwd.length >= 8 &&
    /[0-9]/.test(pwd) &&
    /[a-z]/.test(pwd) &&
    /[A-Z]/.test(pwd) &&
    /[^a-zA-Z0-9]/.test(pwd)
  );
});

// Список критериев с их статусом
const passwordCriteria = computed(() => {
  const pwd = passwordValue.value;
  return [
    { text: 'Длина не менее 8', valid: pwd.length >= 8 },
    { text: 'Цифры', valid: /[0-9]/.test(pwd) },
    { text: 'Буквы нижнего регистра', valid: /[a-z]/.test(pwd) },
    { text: 'Буквы верхнего регистра', valid: /[A-Z]/.test(pwd) },
    { text: 'Спецсимволы', valid: /[^a-zA-Z0-9]/.test(pwd) }
  ];
});

// типы подсветки (пустое поле – без рамки)
const emailFieldClass = computed(() => {
  if (!emailValue.value) return '';
  return isEmailValid.value ? 'valid-field' : 'invalid-field';
});
const passwordFieldClass = computed(() => {
  if (!passwordValue.value) return '';
  return isPasswordValid.value ? 'valid-field' : 'invalid-field';
});

// активность кнопки
const isFormValid = computed(() => {
  return isEmailValid.value && isPasswordValid.value && agreed.value === true;
});

// Отправка
const onSubmit = () => {
  alert(`Успешная регистрация!\nEmail: ${emailValue.value}\nПароль: ${passwordValue.value}`);
};
</script>

<style scoped>
.registration-form {
  max-width: 500px;
  margin: 0 auto;
  padding: 20px;
  font-family: Arial, sans-serif;
  border: 1px solid #ccc;
  border-radius: 8px;
  background: #f9f9f9;
}
.field-group {
  margin-bottom: 20px;
}
label {
  display: block;
  margin-bottom: 5px;
  font-weight: bold;
}
input[type="email"],
input[type="password"] {
  width: 100%;
  padding: 8px;
  border: 1px solid #ccc;
  border-radius: 4px;
  transition: border-color 0.2s;
  outline: none;
  box-sizing: border-box;
}
.valid-field {
  border-color: green !important;
  border-width: 2px;
}
.invalid-field {
  border-color: red !important;
  border-width: 2px;
}
.password-wrapper {
  position: relative;
  display: flex;
  flex-direction: column;
}
.criteria-list {
  list-style: none;
  margin: 8px 0 0 0;
  padding: 0;
  display: flex;
  flex-wrap: wrap;
  gap: 12px;
}
.criteria-list li {
  font-size: 0.85rem;
  padding: 2px 6px;
  border-radius: 12px;
  background: #eee;
}
.criteria-list li.valid {
  color: green;
  background: #e0ffe0;
}
.criteria-list li.invalid {
  color: red;
  background: #ffe0e0;
}
.checkbox-group label {
  display: flex;
  align-items: center;
  gap: 8px;
  font-weight: normal;
}
button {
  width: 100%;
  padding: 10px;
  background-color: #007bff;
  color: white;
  border: none;
  border-radius: 4px;
  font-size: 16px;
  cursor: pointer;
  transition: background 0.2s;
}
button:disabled {
  background-color: #cccccc;
  cursor: not-allowed;
}
button:not(:disabled):hover {
  background-color: #0056b3;
}
</style>