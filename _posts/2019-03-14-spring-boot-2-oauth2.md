---
layout: post
title: "Implement OAuth2 with Spring Boot 2"
description: "How to configure OAuth2 in Spring Boot 2 setting up the authorization server and the resource server both in the same server"
date: 2019-03-14
comments: true
category: blog
tags: [spring, java, oauth2]
published: true
---

In this chapter I'm going to show how to do a very basic configuration of OAuth2 with spring-boot2. We are setting up the authorization and the resource server in the same server, thus it's easier to show how everything works.
**Spring Boot 2** has become ready in 2018. You can read what's new [in this article](https://www.baeldung.com/new-spring-boot-2)
<!-- more -->

## Authorization Server

To indicate spring that this class contains the Authorization Server configuration we have to use `@EnableAuthorizationServer` annotation. Note that I used the most straightforward and basic configuration possible (only intended for development purposes) to have a server working very quickly.

First, you have to configure the clients that you have to authorize in this server. Each possible client will have a **client** and **secret** credentials. In this example we configure these credentials in memory. Also, we tell spring the grant types authorized for these clients are **password** and **refresh_token**. Note that we could have several different client credentials if our authorization server was aimed to authorize different client applications (i.e. if this was a commercial api, we would allow different applications to connect to our api. Then, we would issue a new pair of client-secret for each of them).

```java
@Override
public void configure(ClientDetailsServiceConfigurer clients) throws Exception {
	clients.inMemory().withClient("spring-sample").secret("{noop}a843jdf8ak").scopes("read", "write")
			.authorizedGrantTypes("password", "refresh_token");
}
```

Similarly to clients credentials, we store the tokens in an In Memory store.

```java
@Bean
public TokenStore tokenStore() {
    return new InMemoryTokenStore();
}
```

Finally, we set the authentication manager by constructing this authorization server from an `AuthenticationManager` passed as parameter. The AuthenticationManager is an important part of all this because it's the responsible for authenticating the clients. We configure what authentication manager to use in `WebSecurityConfiguration`.

```java
@Autowired
public AuthorizationServerConfiguration(AuthenticationManager authenticationManager) {
	this.authenticationManager = authenticationManager;
}
```

Below is the whole class

```java
@Configuration
@EnableAuthorizationServer
public class AuthorizationServerConfiguration extends AuthorizationServerConfigurerAdapter {

	private AuthenticationManager authenticationManager;

	@Autowired
	public AuthorizationServerConfiguration(AuthenticationManager authenticationManager) {
		this.authenticationManager = authenticationManager;
	}

	@Override
	public void configure(AuthorizationServerEndpointsConfigurer configurer) {
		configurer.tokenStore(tokenStore());
		configurer.authenticationManager(authenticationManager);
	}

	@Override
	public void configure(ClientDetailsServiceConfigurer clients) throws Exception {
		clients.inMemory().withClient("spring-sample").secret("{noop}a843jdf8ak").scopes("read", "write")
				.authorizedGrantTypes("password", "refresh_token");
	}

	@Bean
	public TokenStore tokenStore() {
		return new InMemoryTokenStore();
	}

	/**
	 * Enables calling of /oauth/check_token to all, remove to disable
	 */
	@Override
	public void configure(AuthorizationServerSecurityConfigurer oauthServer) {
		oauthServer.checkTokenAccess("permitAll()");
	}

}
```


## Web Security Configuration

First important thing is that we have to tell Spring that web security configuration is enabled by adding the `@EnableWebSecurity` annotation at class level.

As we did with the authorization server, we made a very basic configuration of WebSecurityConfiguration. Tipically, the user details are configured separately by implementing `UserDetailsService` interface and obtaining them from a database. But in this case, for demo purposes we have configured the user details in memory by using `InMemoryUserDetailsManager` which is a pre-built class which implements `UserDetailsService` service interface.

```java
@EnableWebSecurity
public class WebSecurityConfiguration extends WebSecurityConfigurerAdapter {

	/**
	 * Enables password grant_type
	 */
	@Bean
	@Override
	public AuthenticationManager authenticationManagerBean() throws Exception {
		return super.authenticationManagerBean();
	}

	@Bean
	@Override
	public UserDetailsService userDetailsServiceBean() throws Exception {
		PasswordEncoder encoder = PasswordEncoderFactories.createDelegatingPasswordEncoder();
		return new InMemoryUserDetailsManager(
				User.withUsername("enduser").password(encoder.encode("password")).roles("USER").build());
	}

}
```

## Resource Server

The resource server is configured by adding `@EnableResourceServer` annotation at class level. See the below example.

```java
@Configuration
@EnableResourceServer
public class ResourceServerConfiguration extends ResourceServerConfigurerAdapter {

	@Override
	public void configure(ResourceServerSecurityConfigurer resources) {
		resources.resourceId("api-resource");
	}

	@Override
	public void configure(HttpSecurity http) throws Exception {
		http.authorizeRequests().antMatchers("/api/v{[0-9]+}/test").authenticated();
	}
}
```

## Build and Run

With all this we have an authorization server up and running. We start the server with the below main class.

```java
@SpringBootApplication
@Configuration
@EnableAuthorizationServer
public class App {
	public static void main(String[] args) {
		SpringApplication.run(App.class, args);
	}
}
```
Build and run the application with the below commands.

```sh
mvn clean install
mvn spring-boot:run
```

## Testing

Finally you can test this stuff using **curl** command. First is to get the access token to have access to the api resources.

```sh
curl -X POST --user "spring-sample:a843jdf8ak" -d "grant_type=password&username=enduser&password=password" http://localhost:8080/oauth/token
```

This command returns something like this:

```json
{
    "access_token": "a98e2a68-89dc-4991-ba6e-c8713f56ec5f",
    "token_type": "bearer",
    "refresh_token": "1c46f591-ed6f-4e0a-93f5-b9811179c028",
    "expires_in": 43199,
    "scope": "read write"
}
```

Then, you can invoke an api resource like this:

```sh
curl --header "Authorization: Bearer a98e2a68-89dc-4991-ba6e-c8713f56ec5f" http://localhost:8080/api/v1/test
```

The response will be similar to this:

```json
{
    "id": "5456806040857622917",
    "time": "2019-03-14 08:07:14.461"
}
```


Feel free to download the source code from [**Github**](https://github.com/rfvallina/spring-boot2-oauth2-sample)
