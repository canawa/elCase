<script setup>
import { ref } from 'vue' // ref - это функция из vue, которая позволяет создавать реактивные переменные (изменив их можно не перезагружать страницу чтобы увидеть результат) 
import {useRouter} from 'vue-router' // импортируем из vue-router, которая позволяет переходить на другую страницу
import RegisterModal from './RegisterModal.vue'
import LoginModal from './LoginModal.vue'
import ForgotPasswordModal from './ForgotPasswordModal.vue'
import ProfileModal from './ProfileModal.vue'
import VoucherModal from './VoucherModal.vue'
const router = useRouter() // это функция из vue-router, которая позволяет переходить на другую страницу
const errorMessage = ref('')
const isLoggedIn = ref(false)
let showRegisterModal = ref(false)
let showLoginModal = ref(false)
let showForgotPasswordModal = ref(false)
let showProfileModal = ref(false)
let showVoucherModal = ref(false)
let userDropdownMenu = ref(false)
let balanceDropdownMenu = ref(false)
let userBalance = ref(0)

const getUserBalance = async () => {
  const token = localStorage.getItem('access_token')
  const response = await fetch('/api/getuserbalance', {
    headers: {
      'Authorization': `Bearer ${token}`
    }
  })
  const data = await response.json()
  console.log(data.data[0])
  userBalance.value = data.data[0].balance + '$'
  // console.log(userBalance.value)
}


const logout = async () => {
  const response = await fetch('/api/logout', {
    method: 'POST',
   
  })
  const data = await response.json()
  console.log(data)
  if (response.ok) {
    console.log('Successfully logged out')
    localStorage.removeItem('access_token')
    location.reload()
  }
  else {
    console.log('Failed to logout')
  }
}



const checkIfLoggedIn = async () => {
  const token = localStorage.getItem('access_token')
  const response = await fetch('/api/checkifloggedin', {
    headers: {
      'Authorization': `Bearer ${token}` // отправляет токен в строке запроса (стандратная форма такая просто удобно и очень безопасно)
    }
  })

  const data = await response.json()

  console.log(data)

  if (data.error) {
    console.log('not logged in')
  }
  else {
    console.log('logged in')
  }

  if (data.user || data.session) {
    isLoggedIn.value = true
    router.push('/')
    getUserBalance() // получаем баланс после успешного логина
  }
  else {
    isLoggedIn.value = false
  }


}
checkIfLoggedIn()

const handleProfile = () => {
  console.log('Profile clicked')
  userDropdownMenu.value = false
  showProfileModal.value = true
}

const handleVouchers = () => {
  console.log('Vouchers clicked')
  userDropdownMenu.value = false
  showVoucherModal.value = true
}

const handleBonuses = () => {
  console.log('Bonuses clicked')
  userDropdownMenu.value = false
}

const handleSettings = () => {
  console.log('Settings clicked')
  userDropdownMenu.value = false
}
const handleBalanceClick = () => {
  balanceDropdownMenu.value = !balanceDropdownMenu.value
  userDropdownMenu.value = false // закрываем другое меню при открытии этого
}

const handleUserIconClick = () => {
  userDropdownMenu.value = !userDropdownMenu.value
  balanceDropdownMenu.value = false // закрываем другое меню при открытии этого
}

const handleDeposit = () => {
  console.log('Deposit clicked')
  balanceDropdownMenu.value = false
  // Здесь можно добавить логику для депозита
}

const handleWithdraw = () => {
  console.log('Withdraw clicked')
  balanceDropdownMenu.value = false
  // Здесь можно добавить логику для вывода средств
}

const handleHistory = () => {
  console.log('History clicked')
  balanceDropdownMenu.value = false
  // Здесь можно добавить логику для просмотра истории
}


</script>

<template>

<div id="header"  @click="showLoginModal = false, showRegisterModal = false" >
    <div id="header-logo">
    
      <img src="/src/assets/logo.png" alt="logo" id="logo" @click="router.push('/')"> <!-- logo - это переменная, которая лежит в App.vue. Переменные из script передаются в template через {{ }} -->
    
    </div>

    <div id="header-logged-in-user" v-if="isLoggedIn"> 
      <div id="user-balance" @click="handleBalanceClick"> <span class="material-icons" id="user-balance-icon"> account_balance_wallet </span> {{ userBalance }} </div>
      <div> <img src="@/assets/user_icon.png" alt="user-icon" id="user-icon" @click="handleUserIconClick" > </div> 

      <div id="user-dropdown-menu" v-if="userDropdownMenu"> 

        <div id="user-dropdown-menu-row" @click="handleProfile()"> Profile</div>
        <div id="user-dropdown-menu-row" @click="handleVouchers()">Vouchers</div>
        <div id="user-dropdown-menu-row" @click="handleBonuses()">Bonuses</div>
        <div id="user-dropdown-menu-row" @click="handleSettings()">Settings</div>
        <div id="user-dropdown-menu-last-row" @click="logout()"> Logout</div>

      </div>

      <div id="balance-dropdown-menu" v-if="balanceDropdownMenu"> 

        <div id="balance-dropdown-menu-row" @click="handleDeposit"> Deposit</div>
        <div id="balance-dropdown-menu-row" @click="handleWithdraw">Withdraw</div>
        <div id="balance-dropdown-menu-last-row" @click="handleHistory">History</div>

      </div>
      
     

     </div>
    <div id="header-auth-buttons" @click.stop v-if="!isLoggedIn" >  <!-- в template vue-js автоматически разворачивает ref, поэтому можно не писать про .value -->
<!-- Не пиши == true или == false, пиши просто !isLoggedIn  или isLoggedIn  потому что vue если написать через === то будет сравнение с ref объектом, а не с булевым значением внутри ref-->
      
      <button type="button" id="header-login-button" @click="showLoginModal = true, showRegisterModal = false">  Login  </button> <!-- в vue js юзаем @click и потом метод, функцию, которая будет выполняться при нажатии на кнопку-->
      <button type="button" id="header-register-button" @click="showRegisterModal = true, showLoginModal = false">  Create Account  </button> <!-- в vue js юзаем @click и потом метод, функцию, которая будет выполняться при нажатии на кнопку-->

    </div>
  </div>  
  <LoginModal v-if="showLoginModal" @closeLogin="showLoginModal = false" @openRegister="showRegisterModal=true, showLoginModal=false" @openForgotPassword="showForgotPasswordModal = true"/>
  <ForgotPasswordModal v-if="showForgotPasswordModal" @closeForgotPassword="showForgotPasswordModal = false" @openLogin="showLoginModal = true, showForgotPasswordModal = false"/>
  <RegisterModal v-if="showRegisterModal" @close="showRegisterModal = false" @openLogin="showLoginModal = true, showRegisterModal = false"/> <!-- этот компонент будет отображаться только если showModal.value === true -->
  <ProfileModal v-if="showProfileModal" @close="showProfileModal = false"/>
  <!-- Если прихоидит от RegisterModal событие через $emit('close') = false то showModal = false -->
<!-- Это метод обмена инфой от дочернего элемента (RegisterModal) к родительскому элементу (header) в котором лежит showModal-->
  <VoucherModal v-if="showVoucherModal" @close="showVoucherModal = false" />

  <div id="error-message">
  
    <p>{{ errorMessage }}</p>
  
  </div>

</template>

<style scoped>


#user-balance {
  display: flex;
  align-items: center;
  justify-content: center;
  background-color: #09a24e;
  font-size: 0.8rem;
  font-weight: 600;
  height: 50%;
  width: 100px;
  border-radius: 0.5rem;
  margin-right: 1rem;
  cursor: pointer;
  transition: background-color 0.3s ease;
}

#user-balance:hover {
  background-color: #29ba67;
}

/* Адаптивные стили для мобильных устройств */
@media (max-width: 768px) {
  #user-balance {
    width: 80px;
    font-size: 0.7rem;
    margin-right: 0.5rem;
    padding: 0.1rem 0.5rem;
  }
  
  #user-balance-icon {
    font-size: 1rem;
    margin-right: 0.6rem;
  }

  #balance-dropdown-menu {
    margin-left: -40px;
    width: 80px;
  }
}

@media (max-width: 480px) {
  #user-balance {
    width: 70px;
    font-size: 0.6rem;
    margin-right: 0.3rem;
    padding: 0.1rem 0.2em;
  }
  
  #user-balance-icon {
    font-size: 1.4rem;
    margin-right: 0.6rem;
  }

  #balance-dropdown-menu {
    margin-left: -35px;
    width: 70px;
  }
}



#user-balance-icon {

  margin-right: 6%;
}

#user-dropdown-menu-row { /* стили для каждой строки выпадающего меню кроме последней*/
  display: flex;
  align-items: center;
  justify-content: center;
  color: #767678;
  font-size: 0.8rem;
  font-weight: 600;
  height: 19.5%;
  width: 100%;
  border-bottom: 1px solid #767678;
  cursor: pointer;


}

#user-dropdown-menu-last-row { /* стили для каждой строки выпадающего меню кроме последней*/
  display: flex;
  align-items: center;
  justify-content: center;
  color: #767678;
  font-size: 0.8rem;
  font-weight: 600;
  height: 19.5%;
  width: 100%;
  cursor: pointer;
}

#user-dropdown-menu-last-row:hover {
  background-color: #76767836;
}

#user-dropdown-menu-row:hover {
  background-color: #76767836;
}


#user-dropdown-menu {
  position: absolute;
  margin-top: 190px;
  background-color: whitesmoke;
  border-radius: 0.5rem;
  height: 150px;
  width: 100px;
  box-shadow: 0 0 10px 0 rgba(0, 0, 0, 0.5); /* тень 0 смещение по оси x, 0 смещение по оси y, 10px blur, 0 растяжение тени по ширине (spread), это смещение! */
}

#balance-dropdown-menu {
  position: absolute;
  margin-top: 190px;
  margin-left: -50px;
  background-color: whitesmoke;
  border-radius: 0.5rem;
  height: 120px;
  width: 100px;
  box-shadow: 0 0 10px 0 rgba(0, 0, 0, 0.5);
}

#balance-dropdown-menu-row {
  display: flex;
  align-items: center;
  justify-content: center;
  color: #767678;
  font-size: 0.8rem;
  font-weight: 600;
  height: 33.33%;
  width: 100%;
  border-bottom: 1px solid #767678;
  cursor: pointer;
}

#balance-dropdown-menu-last-row {
  display: flex;
  align-items: center;
  justify-content: center;
  color: #767678;
  font-size: 0.8rem;
  font-weight: 600;
  height: 33.33%;
  width: 100%;
  cursor: pointer;
}

#balance-dropdown-menu-row:hover {
  background-color: #76767836;
}

#balance-dropdown-menu-last-row:hover {
  background-color: #76767836;
}

#user-icon {
  height: 4vh; /* 2% от высоты экрана, как у логотипа */
  margin-right: 0.8rem;
  cursor: pointer;
  transition: transform 0.3s ease, filter 0.3s ease;
}

#user-icon:hover {
  transform: scale(1.1);
  filter: brightness(1.2);
}

#logo {
  width: 10vh; /* 10vh - это 10% от высоты экрана (ДА ВСЕ ПРАВИЛЬНО, ВЫСОТЫ ДЛЯ ШИРИНЫ, ПОТОМУ ЧТО НИКТО НЕ СЖИМАЕТ ЭКРАН ПО ВЕРТИКАЛИ)*/
  margin-right: 1rem;
  cursor: pointer;
} 



#header { /* стили для div с id logo */
  display: flex;
  color: white;
  width: 100%;
  height: 8%;
  font-family: 'Poppins', sans-serif;
  font-weight: 600;
  align-items: center;
  justify-content: space-between;
  background-color: #12192b;
  border-radius: 1rem;
}

#header-logged-in-user {
  display: flex;
  align-items: center;
  justify-content: right;
  flex-direction: row;
 
  width: 8%;
  height: 100%;
}

/* Адаптивные стили для мобильных устройств */
@media (max-width: 768px) {
  #header-logged-in-user {
    width: auto;
    min-width: 120px;
  }
}

@media (max-width: 480px) {
  #header-logged-in-user {
    width: auto;
    min-width: 100px;
  }
}

#header-auth-buttons {
  display: flex;
  align-items: center;
  justify-content: right;
  font-size: 1rem;
  margin-right: 1.5%;
  width: 15%;
  height: 100%;
  /* border: 2px solid white; */
}


#header-register-button {
  display: flex;
  align-items: center;
  cursor: pointer;
  color: white;
  background-color: #09a24e;
  transition: background-color 0.3s ease;
  border-radius: 0.5rem;
  height: 4vh;
  border: none;
  box-shadow: 0 0 5px 0 rgba(0, 0, 0, 0.5);
  
}

#header-register-button:hover {
  background-color: #29ba67;
}

#header-login-button {
  display: flex;
  align-items: center;
  cursor: pointer;
  color: white;
  background-color: #1e2841;
  transition: background-color 0.3s ease;
  box-shadow: 0 0 5px 0 rgba(0, 0, 0, 0.5);
  border-radius: 0.5rem;
  height: 4vh;
  padding: 0 1vh;
  margin-right: 1vh;
  border: none;
}



</style>