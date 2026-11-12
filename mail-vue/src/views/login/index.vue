<template>
  <div id="login-box" :style=" background ? 'background: var(--el-bg-color)' : ''" v-loading="oauthLoading" element-loading-text="登录中...">
    <div class="starfield" v-if="!settingStore.settings.background" aria-hidden="true">
      <canvas ref="starCanvas" class="star-canvas"></canvas>
      <div class="nebula nebula-1"></div>
      <div class="nebula nebula-2"></div>
      <div class="star-grid"></div>
    </div>
    <div v-else :style="background"></div>
    <div class="form-wrapper">
      <div class="container">
        <span class="form-title">{{ settingStore.settings.title }}</span>
        <span class="form-desc" v-if="show === 'login'">{{ $t('loginTitle') }}</span>
        <span class="form-desc" v-else>{{ $t('regTitle') }}</span>
        <div v-show="show === 'login'">
          <el-input :class="settingStore.settings.loginDomain === 0 ? 'email-input' : ''" v-model="form.email"
                    type="text" :placeholder="$t('emailAccount')" autocomplete="off">
            <template #append v-if="settingStore.settings.loginDomain === 0">
              <div @click.stop="openSelect">
                <el-select
                    v-if="show === 'login'"
                    ref="mySelect"
                    v-model="suffix"
                    :placeholder="$t('select')"
                    class="select"
                >
                  <el-option
                      v-for="item in domainList"
                      :key="item"
                      :label="item"
                      :value="item"
                  />
                </el-select>
                <div style="color: var(--el-text-color-primary)">
                  <span>{{ suffix }}</span>
                  <Icon class="setting-icon" icon="mingcute:down-small-fill" width="20" height="20"/>
                </div>
              </div>
            </template>
          </el-input>
          <el-input v-model="form.password" :placeholder="$t('password')" type="password" autocomplete="off">
          </el-input>
          <el-button class="btn" type="primary" @click="submit" :loading="loginLoading"
          >{{ $t('loginBtn') }}
          </el-button>
          <el-button class="btn" v-if="settingStore.settings.linuxdoSwitch"  style="margin-top: 10px"  @click="linuxDoLogin">
            <el-avatar src="/image/linuxdo.webp" :size="18" style="margin-right: 10px" />LinuxDo
          </el-button>
        </div>
        <div v-show="show !== 'login'">
          <el-input class="email-input" v-model="registerForm.email" type="text" :placeholder="$t('emailAccount')"
                    autocomplete="off">
            <template #append>
              <div @click.stop="openSelect">
                <el-select
                    v-if="show !== 'login'"
                    ref="mySelect"
                    v-model="suffix"
                    :placeholder="$t('select')"
                    class="select"
                >
                  <el-option
                      v-for="item in domainList"
                      :key="item"
                      :label="item"
                      :value="item"
                  />
                </el-select>
                <div>
                  <span>{{ suffix }}</span>
                  <Icon class="setting-icon" icon="mingcute:down-small-fill" width="20" height="20"/>
                </div>
              </div>
            </template>
          </el-input>
          <el-input v-model="registerForm.password" :placeholder="$t('password')" type="password" autocomplete="off"/>
          <el-input v-model="registerForm.confirmPassword" :placeholder="$t('confirmPwd')" type="password"
                    autocomplete="off"/>
          <el-input v-if="settingStore.settings.regKey === 0" v-model="registerForm.code" :placeholder="$t('regKey')"
                    type="text" autocomplete="off"/>
          <el-input v-if="settingStore.settings.regKey === 2" v-model="registerForm.code"
                    :placeholder="$t('regKeyOptional')" type="text" autocomplete="off"/>
          <div v-show="verifyShow"
               class="register-turnstile"
               :data-sitekey="settingStore.settings.siteKey"
               data-callback="onTurnstileSuccess"
               data-error-callback="onTurnstileError"
               data-after-interactive-callback="loadAfter"
               data-before-interactive-callback="loadBefore"
          >
            <span style="font-size: 12px;color: #F56C6C" v-if="botJsError">{{ $t('verifyModuleFailed') }}</span>
          </div>
          <el-button class="btn" style="margin: 0" type="primary" @click="submitRegister" :loading="registerLoading"
          >{{ $t('regBtn') }}
          </el-button>
          <el-button v-if="settingStore.settings.linuxdoSwitch" class="btn" style="margin-top: 10px"  @click="linuxDoLogin">
            <el-avatar src="/image/linuxdo.webp" :size="18" style="margin-right: 10px" />LinuxDo
          </el-button>
        </div>
        <template v-if="settingStore.settings.register === 0">
          <div class="switch" @click="show = 'register'" v-if="show === 'login'">{{ $t('noAccount') }}
            <span>{{ $t('regSwitch') }}</span></div>
          <div class="switch" @click="show = 'login'" v-else>{{ $t('hasAccount') }} <span>{{ $t('loginSwitch') }}</span>
          </div>
        </template>
      </div>
    </div>
    <el-dialog class="bind-dialog" v-model="showBindForm"  title="注册邮箱" >
      <div class="bind-container">
        <el-input v-model="bindForm.email" type="text" :placeholder="$t('emailAccount')" autocomplete="off">
          <template #append>
            <div @click.stop="openSelect">
              <el-select
                  ref="mySelect"
                  v-model="suffix"
                  :placeholder="$t('select')"
                  class="select"
              >
                <el-option
                    v-for="item in domainList"
                    :key="item"
                    :label="item"
                    :value="item"
                />
              </el-select>
              <div>
                <span>{{ suffix }}</span>
                <Icon class="setting-icon" icon="mingcute:down-small-fill" width="20" height="20"/>
              </div>
            </div>
          </template>
        </el-input>
        <el-input v-if="settingStore.settings.regKey === 0" v-model="bindForm.code" :placeholder="$t('regKey')"
                  type="text" autocomplete="off"/>
        <el-input v-if="settingStore.settings.regKey === 2" v-model="bindForm.code"
                  :placeholder="$t('regKeyOptional')" type="text" autocomplete="off"/>
        <el-button class="btn" type="primary" @click="bind" :loading="bindLoading"
        >绑定
        </el-button>
      </div>
    </el-dialog>
    <a v-show="settingStore.settings.projectLink" class="github" href="https://github.com/AndrewYukon/cloud-mail-plus">
      <Icon icon="mingcute:github-line" color="#1890ff" width="20" height="20" />
    </a>
  </div>
</template>

<script setup>
import router from "@/router";
import {computed, nextTick, onMounted, onUnmounted, reactive, ref} from "vue";
import {login} from "@/request/login.js";
import {register} from "@/request/login.js";
import {isEmail} from "@/utils/verify-utils.js";
import {useSettingStore} from "@/store/setting.js";
import {useAccountStore} from "@/store/account.js";
import {useUserStore} from "@/store/user.js";
import {useUiStore} from "@/store/ui.js";
import {Icon} from "@iconify/vue";
import {cvtR2Url} from "@/utils/convert.js";
import {loginUserInfo} from "@/request/my.js";
import {permsToRouter} from "@/perm/perm.js";
import {useI18n} from "vue-i18n";
import {oauthBindUser, oauthLinuxDoLogin} from "@/request/ouath.js";

const {t} = useI18n();
const accountStore = useAccountStore();
const userStore = useUserStore();
const uiStore = useUiStore();
const settingStore = useSettingStore();
const loginLoading = ref(false)
const bindLoading = ref(false)
const oauthLoading = ref(false);
const showBindForm = ref(false);
const show = ref('login')

const bindForm = reactive({
  email: '',
  oauthUserId: '',
  code: ''
})

const form = reactive({
  email: '',
  password: '',

});
const mySelect = ref()
const suffix = ref('')
const registerForm = reactive({
  email: '',
  password: '',
  confirmPassword: '',
  code: null
})
const domainList = settingStore.domainList;
const registerLoading = ref(false)
suffix.value = domainList[0]
const verifyShow = ref(false)
let verifyToken = ''
let turnstileId = null
let botJsError = ref(false)
let verifyErrorCount = 0

// ---- 科幻星空：Canvas 星星 + 流星，纯前端、无依赖 ----
// 注意：Turnstile 仍由 index.html 的 <script src="https://challenges.cloudflare.com/turnstile/v0/api.js">
// 全局加载（window.turnstile），这里不 import 任何本地 turnstile 文件，避免构建报 ENOENT。
const starCanvas = ref(null)
let starRaf = 0
let starList = []
let meteorList = []
let starCtx = null
let starResize = null

onUnmounted(() => {
  cancelAnimationFrame(starRaf)
  if (starResize) window.removeEventListener('resize', starResize)
})

function initStarfield() {
  const canvas = starCanvas.value
  if (!canvas) return
  const reduceMotion = window.matchMedia('(prefers-reduced-motion: reduce)').matches
  const dpr = Math.min(window.devicePixelRatio || 1, 2)
  const resize = () => {
    const w = canvas.clientWidth || window.innerWidth
    const h = canvas.clientHeight || window.innerHeight
    canvas.width = Math.floor(w * dpr)
    canvas.height = Math.floor(h * dpr)
    starCtx = canvas.getContext('2d')
    starCtx.setTransform(dpr, 0, 0, dpr, 0, 0)
    const count = Math.min(220, Math.max(90, Math.floor((w * h) / 9000)))
    const colors = ['#ffffff', '#a5d8ff', '#c4b5fd', '#7dd3fc', '#bae6fd']
    starList = Array.from({length: count}, () => ({
      x: Math.random() * w,
      y: Math.random() * h,
      r: 0.4 + Math.random() * 1.4,
      baseAlpha: 0.35 + Math.random() * 0.65,
      speed: 0.4 + Math.random() * 1.2,
      phase: Math.random() * Math.PI * 2,
      vx: (Math.random() - 0.5) * 0.05,
      vy: 0.02 + Math.random() * 0.06,
      color: colors[Math.floor(Math.random() * colors.length)],
    }))
    if (reduceMotion) drawStars(0, w, h)
  }
  const drawStars = (t, w, h) => {
    if (!starCtx) return
    starCtx.clearRect(0, 0, w, h)
    for (const st of starList) {
      st.x += st.vx
      st.y += st.vy
      if (st.y > h + 2) { st.y = -2; st.x = Math.random() * w }
      if (st.x < -2) st.x = w + 2
      if (st.x > w + 2) st.x = -2
      const alpha = st.baseAlpha * (0.55 + 0.45 * Math.sin(t * 0.001 * st.speed + st.phase))
      starCtx.globalAlpha = Math.max(0.08, alpha)
      starCtx.fillStyle = st.color
      starCtx.beginPath()
      starCtx.arc(st.x, st.y, st.r, 0, Math.PI * 2)
      starCtx.fill()
      // 亮星加十字光晕
      if (st.r > 1.4) {
        starCtx.globalAlpha = alpha * 0.35
        starCtx.fillRect(st.x - st.r * 3, st.y - 0.5, st.r * 6, 1)
        starCtx.fillRect(st.x - 0.5, st.y - st.r * 3, 1, st.r * 6)
      }
    }
    starCtx.globalAlpha = 1
    // 流星：每帧小概率生成，最多同时 2 颗
    if (!reduceMotion && Math.random() < 0.003 && meteorList.length < 2 && w > 500) {
      meteorList.push({x: w * (0.3 + Math.random() * 0.7), y: h * Math.random() * 0.25, vx: -(5 + Math.random() * 3), vy: 2.5 + Math.random() * 1.5, life: 1})
    }
    meteorList = meteorList.filter(m => m.life > 0)
    for (const m of meteorList) {
      m.x += m.vx; m.y += m.vy; m.life -= 0.016
      const grad = starCtx.createLinearGradient(m.x, m.y, m.x - m.vx * 12, m.y - m.vy * 12)
      grad.addColorStop(0, `rgba(165, 216, 255, ${0.9 * m.life})`)
      grad.addColorStop(1, 'rgba(165, 216, 255, 0)')
      starCtx.strokeStyle = grad
      starCtx.lineWidth = 1.6
      starCtx.beginPath()
      starCtx.moveTo(m.x, m.y)
      starCtx.lineTo(m.x - m.vx * 12, m.y - m.vy * 12)
      starCtx.stroke()
    }
  }
  const loop = (t) => {
    drawStars(t, canvas.clientWidth || window.innerWidth, canvas.clientHeight || window.innerHeight)
    if (!reduceMotion) starRaf = requestAnimationFrame(loop)
  }
  resize()
  starResize = resize
  window.addEventListener('resize', resize)
  if (!reduceMotion) starRaf = requestAnimationFrame(loop)
}

onMounted(() => {
  initStarfield()
})

window.onTurnstileSuccess = (token) => {
  verifyToken = token;
};

window.onTurnstileError = (e) => {
  if (verifyErrorCount >= 4) {
    return
  }
  verifyErrorCount++
  console.warn('人机验加载失败', e)
  setTimeout(() => {
    nextTick(() => {
      if (!turnstileId) {
        turnstileId = window.turnstile.render('.register-turnstile')
      } else {
        window.turnstile.reset(turnstileId);
      }
    })
  }, 1500)
};

window.loadAfter = (e) => {
  console.log('loadAfter')
}

window.loadBefore = (e) => {
  console.log('loadBefore')
}

// 星空下登录卡用深色玻璃底：alpha 仍沿用后台“登录透明度”，下限 0.55 保证文字可读
const loginCardBg = computed(() => {
  const opacity = Number(settingStore.settings.loginOpacity ?? 0.88)
  const alpha = Math.min(0.96, Math.max(0.55, opacity))
  return `rgba(8, 14, 32, ${alpha})`
})

const background = computed(() => {

  return settingStore.settings.background ? {
    'background-image': `url(${cvtR2Url(settingStore.settings.background)})`,
    'background-repeat': 'no-repeat',
    'background-size': 'cover',
    'background-position': 'center'
  } : ''
})

const openSelect = () => {
  mySelect.value.toggleMenu()
}

function linuxDoLogin() {
  const clientId = settingStore.settings.linuxdoClientId
  const redirectUri = encodeURIComponent(settingStore.settings.linuxdoCallbackUrl)
  window.location.href =
      `https://connect.linux.do/oauth2/authorize?client_id=${clientId}&redirect_uri=${redirectUri}&response_type=code&scope=openid+profile+email`
}

linuxDoGetUser();

async function linuxDoGetUser() {

  const params = new URLSearchParams(window.location.search)
  const code = params.get('code')

  if (code) {

    oauthLoading.value = true
    oauthLinuxDoLogin(code).then(data => {

      bindForm.oauthUserId = data.userInfo.oauthUserId;

      if (!data.token) {
        showBindForm.value = true
        oauthLoading.value = false
        ElMessage({
          message: '请注册绑定一个邮箱',
          type: 'warning',
          duration: 4000,
          plain: true,
        })
        return;
      }

      saveToken(data.token);
    }).catch(() => {
      oauthLoading.value = false
    })
  }

  const cleanUrl = window.location.origin + window.location.pathname
  window.history.replaceState({}, '', cleanUrl)
}

function bind() {

  if (!bindForm.email) {
    ElMessage({
      message: t('emptyEmailMsg'),
      type: 'error',
      plain: true,
    })
    return
  }


  if (bindForm.email.length < settingStore.settings.minEmailPrefix) {
    ElMessage({
      message: t('minEmailPrefix', {msg: settingStore.settings.minEmailPrefix}),
      type: 'error',
      plain: true,
    })
    return
  }

  let email = bindForm.email + suffix.value;


  if (!isEmail(email)) {
    ElMessage({
      message: t('notEmailMsg'),
      type: 'error',
      plain: true,
    })
    return
  }

  if (settingStore.settings.regKey === 0) {

    if (!bindForm.code) {

      ElMessage({
        message: t('emptyRegKeyMsg'),
        type: 'error',
        plain: true,
      })
      return
    }

  }

  const form = {email: bindForm.email + suffix.value, oauthUserId: bindForm.oauthUserId, code: bindForm.code}

  bindLoading.value = true
  oauthBindUser(form).then(data => {
    saveToken(data.token)
  }).catch(() => {
    bindLoading.value = false
  })
}

const submit = () => {

  if (!form.email) {
    ElMessage({
      message: t('emptyEmailMsg'),
      type: 'error',
      plain: true,
    })
    return
  }

  let email = form.email + (settingStore.settings.loginDomain === 0 ? suffix.value : '');

  if (!isEmail(email)) {
    ElMessage({
      message: t('notEmailMsg'),
      type: 'error',
      plain: true,
    })
    return
  }

  if (!form.password) {
    ElMessage({
      message: t('emptyPwdMsg'),
      type: 'error',
      plain: true,
    })
    return
  }

  loginLoading.value = true
  login(email, form.password).then(async data => {
    await saveToken(data.token)
  }).finally(() => {
    loginLoading.value = false
  })
}

async function saveToken(token) {
  localStorage.setItem('token', token)
  const user = await loginUserInfo();
  accountStore.currentAccountId = user.account.accountId;
  accountStore.currentAccount = user.account;
  userStore.user = user;
  const routers = permsToRouter(user.permKeys);
  routers.forEach(routerData => {
    router.addRoute('layout', routerData);
  });
  await router.replace({name: 'layout'})
  uiStore.showNotice()
  oauthLoading.value = false;
  bindLoading.value = false;
}


function submitRegister() {

  if (!registerForm.email) {
    ElMessage({
      message: t('emptyEmailMsg'),
      type: 'error',
      plain: true,
    })
    return
  }

  console.log(registerForm.email)

  if (registerForm.email.length < settingStore.settings.minEmailPrefix) {
    ElMessage({
      message: t('minEmailPrefix', {msg: settingStore.settings.minEmailPrefix}),
      type: 'error',
      plain: true,
    })
    return
  }

  if (!isEmail(registerForm.email + suffix.value)) {
    ElMessage({
      message: t('notEmailMsg'),
      type: 'error',
      plain: true,
    })
    return
  }

  if (!registerForm.password) {
    ElMessage({
      message: t('emptyPwdMsg'),
      type: 'error',
      plain: true,
    })
    return
  }

  if (registerForm.password.length < 6) {
    ElMessage({
      message: t('pwdLengthMsg'),
      type: 'error',
      plain: true,
    })
    return
  }

  if (registerForm.password !== registerForm.confirmPassword) {

    ElMessage({
      message: t('confirmPwdFailMsg'),
      type: 'error',
      plain: true,
    })
    return
  }

  if (settingStore.settings.regKey === 0) {

    if (!registerForm.code) {

      ElMessage({
        message: t('emptyRegKeyMsg'),
        type: 'error',
        plain: true,
      })
      return
    }

  }

  if (!verifyToken && (settingStore.settings.registerVerify === 0 || (settingStore.settings.registerVerify === 2 && settingStore.settings.regVerifyOpen))) {
    if (!verifyShow.value) {
      verifyShow.value = true
      nextTick(() => {
        if (!turnstileId) {
          try {
            turnstileId = window.turnstile.render('.register-turnstile')
          } catch (e) {
            botJsError.value = true
            console.log('人机验证js加载失败')
          }
        } else {
          window.turnstile.reset('.register-turnstile')
        }
      })
    } else if (!botJsError.value) {
      ElMessage({
        message: t('botVerifyMsg'),
        type: "error",
        plain: true
      })
    }
    return;
  }

  registerLoading.value = true

  const form = {
    email: registerForm.email + suffix.value,
    password: registerForm.password,
    token: verifyToken,
    code: registerForm.code
  }

  register(form).then(({regVerifyOpen}) => {
    show.value = 'login'
    registerForm.email = ''
    registerForm.password = ''
    registerForm.confirmPassword = ''
    registerForm.code = ''
    registerLoading.value = false
    verifyToken = ''
    settingStore.settings.regVerifyOpen = regVerifyOpen
    verifyShow.value = false
    ElMessage({
      message: t('regSuccessMsg'),
      type: 'success',
      plain: true,
    })
  }).catch(res => {

    registerLoading.value = false

    if (res.code === 400) {
      verifyToken = ''
      settingStore.settings.regVerifyOpen = true
      if (turnstileId) {
        window.turnstile.reset(turnstileId)
      } else {
        nextTick(() => {
          turnstileId = window.turnstile.render('.register-turnstile')
        })
      }
      verifyShow.value = true

    }
  });
}

</script>


<style>
.el-select-dropdown__item {
  padding: 0 15px;
}

.no-autofill-pwd {
  .el-input__inner {
    -webkit-text-security: disc !important;
  }
}
</style>

<style lang="scss" scoped>

.form-wrapper {
  position: fixed;
  inset: 0;
  z-index: 10;
  display: flex;
  align-items: center;
  justify-content: center;
  padding: 20px;
  pointer-events: none;
}

.container {
  background: v-bind(loginCardBg);
  backdrop-filter: blur(14px);
  -webkit-backdrop-filter: blur(14px);
  padding: 36px 40px;
  display: flex;
  flex-direction: column;
  justify-content: center;
  width: min(420px, 100%);
  height: auto;
  pointer-events: auto;
  color: #eaf2ff;
  border: 1px solid rgba(125, 180, 255, 0.28);
  border-radius: 20px;
  box-shadow:
    0 20px 60px rgba(0, 0, 0, 0.55),
    0 0 0 1px rgba(255, 255, 255, 0.04),
    0 0 44px rgba(56, 130, 246, 0.18);
  /* 小幅浮动：上下 8px，6 秒一圈，慢而稳 */
  animation: login-float 6s ease-in-out infinite;
  will-change: transform;
  position: relative;
  overflow: hidden;
  @media (max-width: 1024px) {
    padding: 32px 28px;
  }
  @media (max-width: 767px) {
    padding: 28px 20px;
    border-radius: 16px;
  }
  /* 顶部一条科幻辉光线 */
  &::before {
    content: "";
    position: absolute;
    top: 0;
    left: 12%;
    right: 12%;
    height: 2px;
    background: linear-gradient(90deg, transparent, #38bdf8, #a78bfa, transparent);
    opacity: 0.9;
  }

  .btn {
    height: 36px;
    width: 100%;
    border-radius: 6px;
  }

  .form-desc {
    margin-top: 5px;
    margin-bottom: 18px;
    color: rgba(180, 200, 235, 0.85);
  }

  .form-title {
    font-weight: bold;
    font-size: 22px !important;
    color: #ffffff;
    /* 科幻渐变标题字 */
    background: linear-gradient(90deg, #7dd3fc, #a78bfa 55%, #67e8f9);
    -webkit-background-clip: text;
    background-clip: text;
    -webkit-text-fill-color: transparent;
  }

  .switch {
    margin-top: 20px;
    text-align: center;
    color: rgba(180, 200, 235, 0.85);

    span {
      color: #7dd3fc;
      cursor: pointer;
    }
  }

  :deep(.el-input__wrapper) {
    border-radius: 6px;
    background: var(--el-bg-color);
  }

  .email-input :deep(.el-input__wrapper) {
    border-radius: 6px 0 0 6px;
    background: var(--el-bg-color);
  }

  .el-input {
    height: 38px;
    width: 100%;
    margin-bottom: 18px;

    :deep(.el-input__inner) {
      height: 36px;
    }
  }
}

:deep(.el-select-dropdown__item) {
  padding: 0 10px;
}

:deep(.bind-dialog) {
  width: 400px !important;
  @media (max-width: 440px) {
    width: calc(100% - 40px) !important;
    margin-right: 20px !important;
    margin-left: 20px !important;
  }
}

.bind-container {
  display: grid;
  grid-template-columns: 1fr;
  gap: 15px;
}

.setting-icon {
  position: relative;
  top: 6px;
}

.github {
  position: fixed;
  width: 35px;
  height: 35px;
  display: flex;
  justify-content: center;
  align-items: center;
  border-radius: 50%;
  background: rgba(8, 14, 32, 0.65);
  backdrop-filter: blur(8px);
  bottom: 10px;
  right: 10px;
  z-index: 1000;
  border: 1px solid var(--el-border-color-light);
  box-shadow: var(--el-box-shadow-light);
  cursor: pointer;
}

:deep(.el-input-group__append) {
  padding: 0 !important;
  padding-left: 8px !important;
  padding-right: 4px !important;
  background: var(--el-bg-color);
  border-radius: 0 8px 8px 0;
}

:deep(.el-button+.el-button) {
  margin: 0;
}

.register-turnstile {
  margin-bottom: 18px;
}

.select {
  position: absolute;
  right: 30px;
  width: 100px;
  opacity: 0;
  pointer-events: none;
}

.custom-style {
  margin-bottom: 10px;
}

.custom-style .el-segmented {
  --el-border-radius-base: 6px;
  width: 180px;
}


#login-box {
  position: relative;
  background:
    radial-gradient(ellipse 80% 55% at 18% 8%, rgba(99, 102, 241, 0.32), transparent 60%),
    radial-gradient(ellipse 70% 50% at 85% 90%, rgba(56, 189, 248, 0.20), transparent 60%),
    radial-gradient(ellipse 55% 40% at 78% 12%, rgba(168, 85, 247, 0.16), transparent 65%),
    linear-gradient(180deg, #020617 0%, #070d24 45%, #0d1030 78%, #120a2e 100%);
  font: 100% Arial, sans-serif;
  height: 100%;
  margin: 0;
  padding: 0;
  overflow-x: hidden;
  display: grid;
  grid-template-columns: 1fr;
}


/* ---- 科幻星空层（Canvas 画星星/流星，CSS 做星云和微光） ---- */
.starfield {
  position: absolute;
  inset: 0;
  z-index: 0;
  overflow: hidden;
  pointer-events: none;
}

.star-canvas {
  position: absolute;
  inset: 0;
  width: 100%;
  height: 100%;
  display: block;
}

/* 两团缓慢漂移的星云 */
.nebula {
  position: absolute;
  border-radius: 50%;
  filter: blur(70px);
  opacity: 0.55;
}
.nebula-1 {
  width: 52vw;
  height: 52vw;
  left: -12vw;
  top: -18vw;
  background: radial-gradient(circle, rgba(99, 102, 241, 0.45), transparent 65%);
  animation: nebula-drift 26s ease-in-out infinite alternate;
}
.nebula-2 {
  width: 44vw;
  height: 44vw;
  right: -10vw;
  bottom: -16vw;
  background: radial-gradient(circle, rgba(34, 211, 238, 0.35), transparent 65%);
  animation: nebula-drift 32s ease-in-out infinite alternate-reverse;
}
@keyframes nebula-drift {
  from { transform: translate(0, 0) scale(1); }
  to { transform: translate(4vw, 3vh) scale(1.12); }
}

/* 细密的 CSS 星点打底（Canvas 未启动时也有星空），带闪烁 */
.star-grid {
  position: absolute;
  inset: 0;
  background-image:
    radial-gradient(1px 1px at 12% 22%, rgba(255, 255, 255, 0.9), transparent),
    radial-gradient(1px 1px at 28% 68%, rgba(165, 216, 255, 0.8), transparent),
    radial-gradient(1.5px 1.5px at 44% 12%, rgba(255, 255, 255, 0.85), transparent),
    radial-gradient(1px 1px at 61% 38%, rgba(196, 181, 253, 0.8), transparent),
    radial-gradient(1px 1px at 73% 82%, rgba(255, 255, 255, 0.8), transparent),
    radial-gradient(1.5px 1.5px at 86% 18%, rgba(125, 211, 252, 0.85), transparent),
    radial-gradient(1px 1px at 92% 58%, rgba(255, 255, 255, 0.75), transparent),
    radial-gradient(1px 1px at 5% 84%, rgba(165, 216, 255, 0.7), transparent);
  background-size: 640px 640px;
  animation: star-twinkle 4s ease-in-out infinite alternate;
}
@keyframes star-twinkle {
  from { opacity: 0.45; }
  to { opacity: 1; }
}

/* ---- 登录框小幅浮动：上下 8px ---- */
@keyframes login-float {
  0%, 100% { transform: translateY(0); }
  50% { transform: translateY(-8px); }
}

/* 尊重系统的“减弱动效”设置 */
@media (prefers-reduced-motion: reduce) {
  .container,
  .nebula,
  .star-grid {
    animation: none;
  }
}

</style>
