# Spring Boot reCAPTCHA v3 Starter

[![Maven Central](https://img.shields.io/maven-central/v/com.michael-bull.spring-boot-starter-recaptcha/spring-boot-starter-recaptcha.svg)](https://search.maven.org/search?q=g:com.michael-bull.spring-boot-starter-recaptcha) [![CI Status](https://github.com/michaelbull/spring-boot-starter-recaptcha/workflows/ci/badge.svg)](https://github.com/michaelbull/spring-boot-starter-recaptcha/actions?query=workflow%3Aci) [![License](https://img.shields.io/github/license/michaelbull/spring-boot-starter-recaptcha.svg)](https://github.com/michaelbull/spring-boot-starter-recaptcha/blob/master/LICENSE)

Spring Boot starter for Google's [reCAPTCHA v3][recaptcha-v3].

## Installation

```groovy
repositories {
    mavenCentral()
}

dependencies {
    implementation("com.michael-bull.spring-boot-starter-recaptcha:spring-boot-starter-recaptcha:1.0.2")
}
```

## Getting Started

#### 1. Register reCAPTCHA v3 keys

Register your application on the [key registration page][recaptcha-v3-keys].

#### 2. Add the configuration properties to your `application.yaml`:

```yaml
recaptcha.keys:
  site: "<your site key>"
  secret: "<your secret key>"
```

#### 3. Model the form that recaptcha exists on:

```kotlin
class RegisterForm {

    var recaptchaAction: String? = "register"

    var recaptchaResponseToken: String? = null

    @Email
    var email: String? = null
}
```


#### 4. Add a validator for your form:

```kotlin
@Component
@RequestScope
class RegisterFormValidator @Inject constructor(
    private val request: HttpServletRequest,
    private val recaptchaValidator: RecaptchaValidator
) : Validator {

    override fun supports(clazz: Class<*>): Boolean {
        return RegisterForm::class.java.isAssignableFrom(clazz)
    }

    override fun validate(target: Any, errors: Errors) {
        val form = target as RecoverAccountForm
        val action = form.recaptchaAction
        val responseToken = form.recaptchaResponseToken

        recaptchaValidator
            .validate("recaptchaResponseToken", request, action, responseToken, errors)
            .onSuccess { (_, response) -> checkResponse(response, errors) }
    }

    private fun checkResponse(response: SiteVerifyResponse, errors: Errors) {
        val score = response.score

        if (score != null && score < 0.2) {
            errors.rejectValue("recaptchaResponseToken", "Score too low")
        }
    }
}
```

#### 5. Bind the validator in your `Controller`:

```kotlin
@Controller
class RegisterController @Inject constructor(
    private val formValidator: RegisterFormValidator
) {

    @InitBinder("form")
    fun initFormBinder(binder: WebDataBinder) {
        binder.addValidators(formValidator)
    }

    /* get and post handlers... */
}
```

## I18n

Error codes generated by the RecaptchaValidator can be internationalized by
adding the following entries to your `messages.properties`:

```properties
captcha.error.actionMissing=Captcha action missing.
captcha.error.incomplete=Captcha incomplete.
captcha.error.request=Failed to submit captcha.
captcha.error.responseMissing=No response from captcha service.
captcha.error.response=Error response from captcha service.
captcha.error.failed=Captcha failed. Please try again.
captcha.error.actionMismatch=Captcha action mismatch.
```

## Contributing

Bug reports and pull requests are welcome on [GitHub][github].

## License

This project is available under the terms of the ISC license. See the
[`LICENSE`](LICENSE) file for the copyright information and licensing terms.

[recaptcha-v3]: https://developers.google.com/recaptcha/docs/v3
[recaptcha-v3-keys]: https://g.co/recaptcha/v3
[github]: https://github.com/michaelbull/spring-boot-starter-recaptcha
