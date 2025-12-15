<template>
  <div class="login-container">
    <!-- 背景装饰 -->
    <div class="background-decoration">
      <div class="bg-shape bg-shape-1"></div>
      <div class="bg-shape bg-shape-2"></div>
      <div class="bg-shape bg-shape-3"></div>
    </div>

    <!-- 登录表单 -->
    <el-form ref="form" :model="form" :rules="rules" class="login-form fade-in">
      <!-- 标题 -->
      <div class="login-header">
        <img :src="logoUrl" class="login-logo" alt="图书馆管理系统">
        <h2 class="login-title">图书借阅与管理系统</h2>
        <p class="login-subtitle">欢迎回来，请登录您的账号</p>
      </div>

      <!-- 表单内容 -->
      <el-form-item prop="username">
        <el-input v-model="form.username" placeholder="用户名" clearable class="modern-input">
          <template #prefix>
            <el-icon class="input-icon">
              <User />
            </el-icon>
          </template>
        </el-input>
      </el-form-item>

      <el-form-item prop="password">
        <el-input v-model="form.password" placeholder="密码" clearable show-password class="modern-input">
          <template #prefix>
            <el-icon class="input-icon">
              <Lock />
            </el-icon>
          </template>
        </el-input>
      </el-form-item>

      <el-form-item>
        <div class="captcha-container">
          <el-input v-model="form.validCode" placeholder="请输入验证码" class="modern-input captcha-input">
            <template #prefix>
              <el-icon class="input-icon">
                <Key />
              </el-icon>
            </template>
          </el-input>
          <ValidCode @input="createValidCode" class="captcha-code" />
        </div>
      </el-form-item>

      <el-form-item>
        <el-button type="primary" class="login-button" @click="login">
          <span class="button-text">登录</span>
        </el-button>
      </el-form-item>

      <!-- 底部链接 -->
      <div class="login-footer">
        <el-button type="text" class="footer-link" @click="$router.push('/forget')">
          忘记密码?
        </el-button>
        <el-button type="text" class="footer-link register-link" @click="$router.push('/register')">
          没有账号?前往注册
        </el-button>
      </div>
    </el-form>
  </div>
</template>

<script>
import request from "../utils/request";
import { ElMessage } from "element-plus";
import ValidCode from "../components/Validate";
import { User, Lock, Key } from '@element-plus/icons-vue';

export default {
  name: "Login",
  components: {
    ValidCode,
    User,
    Lock,
    Key
  },
  data() {
    return {
      validCode: '',//通过valicode获取的验证码
      form: {},
      rules: {
        username: [
          {
            required: true,
            message: '请输入用户名',
            trigger: 'blur',
          }
        ],
        password: [
          {
            required: true,
            message: '请输入密码',
            trigger: 'blur',
          }
        ]
      },
      logoUrl: require('../assets/logo.png')
    }
  },
  methods: {
    createValidCode(data) {
      this.validCode = data
    },
    login() {
      this.$refs['form'].validate((valid) => {
        if (valid) {
          if (!this.form.validCode) {
            ElMessage.error("请填写验证码")
            return
          }
          if (this.form.validCode.toLowerCase() !== this.validCode.toLowerCase()) {
            ElMessage.error("验证码错误")
            return
          }

          request.post("user/login", this.form).then(res => {
            if (res.code == 0) {
              ElMessage.success("登录成功")
              sessionStorage.setItem("user", JSON.stringify(res.data))//缓存用户信息
              this.$router.push("/dashboard")
            } else {
              ElMessage.error(res.msg)
            }
          })
        }
      })
    }
  }
}
</script>

<style scoped>
/* 登录容器 */
.login-container {
  position: fixed;
  width: 100%;
  height: 100vh;
  display: flex;
  align-items: center;
  justify-content: center;
  background: var(--bg-primary);
  overflow: hidden;
  z-index: 1;
}

/* 背景装饰 */
.background-decoration {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  overflow: hidden;
  z-index: -1;
}

.bg-shape {
  position: absolute;
  border-radius: 50%;
  background: var(--primary-gradient);
  opacity: 0.1;
  animation: float 20s infinite ease-in-out;
}

.bg-shape-1 {
  width: 600px;
  height: 600px;
  top: -200px;
  left: -100px;
  animation-delay: 0s;
}

.bg-shape-2 {
  width: 400px;
  height: 400px;
  bottom: -100px;
  right: -50px;
  animation-delay: -10s;
}

.bg-shape-3 {
  width: 300px;
  height: 300px;
  top: 50%;
  right: 10%;
  animation-delay: -5s;
}

@keyframes float {

  0%,
  100% {
    transform: translateY(0px) rotate(0deg);
  }

  33% {
    transform: translateY(-20px) rotate(120deg);
  }

  66% {
    transform: translateY(10px) rotate(240deg);
  }
}

/* 登录表单 */
.login-form {
  width: 420px;
  padding: var(--spacing-2xl);
  background: var(--bg-secondary);
  border-radius: var(--radius-xl);
  box-shadow: var(--shadow-xl);
  border: 1px solid var(--border-color);
  backdrop-filter: blur(10px);
  transition: all var(--transition-normal);
}

.login-form:hover {
  box-shadow: 0 25px 50px -12px rgba(0, 0, 0, 0.25);
}

/* 登录头部 */
.login-header {
  text-align: center;
  margin-bottom: var(--spacing-xl);
}

.login-logo {
  width: 80px;
  height: 80px;
  margin-bottom: var(--spacing-lg);
  border-radius: 50%;
  background: var(--primary-gradient);
  padding: var(--spacing-md);
  box-shadow: var(--shadow-lg);
  transition: transform var(--transition-normal);
}

.login-logo:hover {
  transform: scale(1.05);
}

.login-title {
  font-size: 24px;
  font-weight: 600;
  color: var(--text-primary);
  margin-bottom: var(--spacing-xs);
}

.login-subtitle {
  font-size: 14px;
  color: var(--text-secondary);
  margin: 0;
}

/* 现代化输入框 */
.modern-input {
  margin-bottom: var(--spacing-md);
}

.modern-input .el-input__wrapper {
  border-radius: var(--radius-md);
  box-shadow: none;
  border: 1px solid var(--border-color);
  transition: all var(--transition-normal);
}

.modern-input .el-input__wrapper:focus-within {
  box-shadow: 0 0 0 3px rgba(102, 126, 234, 0.1);
  border-color: var(--primary-color);
  transform: translateY(-1px);
}

.modern-input .el-input__inner {
  height: 48px;
  font-size: 14px;
  border-radius: var(--radius-md);
}

.input-icon {
  color: var(--text-muted);
  font-size: 18px;
}

/* 验证码容器 */
.captcha-container {
  display: flex;
  gap: var(--spacing-md);
  align-items: center;
}

.captcha-input {
  flex: 1;
}

.captcha-code {
  flex-shrink: 0;
  width: 120px;
  height: 48px;
  border-radius: var(--radius-md);
  overflow: hidden;
  cursor: pointer;
  transition: all var(--transition-normal);
  border: 1px solid var(--border-color);
}

.captcha-code:hover {
  transform: translateY(-1px);
  box-shadow: var(--shadow-md);
}

/* 登录按钮 */
.login-button {
  width: 100%;
  height: 48px;
  font-size: 16px;
  font-weight: 600;
  border-radius: var(--radius-md);
  background: var(--primary-gradient);
  border: none;
  transition: all var(--transition-normal);
  margin-top: var(--spacing-md);
}

.login-button:hover {
  opacity: 0.9;
  transform: translateY(-2px);
  box-shadow: var(--shadow-lg);
  background: var(--primary-gradient);
}

.login-button:active {
  transform: translateY(0);
}

.button-text {
  letter-spacing: 1px;
}

/* 登录底部链接 */
.login-footer {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-top: var(--spacing-lg);
  padding-top: var(--spacing-lg);
  border-top: 1px solid var(--border-color);
}

.footer-link {
  font-size: 14px;
  color: var(--text-secondary);
  transition: color var(--transition-normal);
  padding: 0;
}

.footer-link:hover {
  color: var(--primary-color);
}

.register-link {
  color: var(--primary-color);
  font-weight: 500;
}

.register-link:hover {
  color: var(--primary-dark);
}

/* 淡入动画 */
.fade-in {
  animation: fadeIn var(--transition-slow);
}

@keyframes fadeIn {
  from {
    opacity: 0;
    transform: translateY(30px);
  }

  to {
    opacity: 1;
    transform: translateY(0);
  }
}

/* 响应式设计 */
@media (max-width: 768px) {
  .login-form {
    width: 90%;
    max-width: 400px;
    padding: var(--spacing-xl);
    margin: 0 var(--spacing-md);
  }

  .login-title {
    font-size: 20px;
  }

  .login-logo {
    width: 60px;
    height: 60px;
  }

  .captcha-container {
    flex-direction: column;
    align-items: stretch;
  }

  .captcha-code {
    width: 100%;
  }

  .bg-shape-1 {
    width: 400px;
    height: 400px;
  }

  .bg-shape-2 {
    width: 300px;
    height: 300px;
  }

  .bg-shape-3 {
    width: 200px;
    height: 200px;
  }
}

@media (max-width: 480px) {
  .login-form {
    padding: var(--spacing-lg);
  }

  .login-title {
    font-size: 18px;
  }

  .login-subtitle {
    font-size: 13px;
  }
}
</style>