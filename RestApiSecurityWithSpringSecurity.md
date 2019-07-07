# Authentication For REST API Services


## 	Create HttpBasic and disable csrf and loginForm 


		@Configuration
		public class SpringSecurityConfig extends WebSecurityConfigurerAdapter {

			// Create 2 users for demo
			@Override
			protected void configure(AuthenticationManagerBuilder auth) throws Exception {

				auth.inMemoryAuthentication()
						.withUser("user").password("{noop}password").roles("USER")
						.and()
						.withUser("admin").password("{noop}password").roles("USER", "ADMIN");

			}

			// Secure the endpoins with HTTP Basic authentication
			@Override
			protected void configure(HttpSecurity http) throws Exception {

				http
						//HTTP Basic authentication
						.httpBasic()
						.and()
						.authorizeRequests()
						.antMatchers(HttpMethod.GET, "/books/**").hasRole("USER")
						.antMatchers(HttpMethod.POST, "/books").hasRole("ADMIN")
						.antMatchers(HttpMethod.PUT, "/books/**").hasRole("ADMIN")
						.antMatchers(HttpMethod.PATCH, "/books/**").hasRole("ADMIN")
						.antMatchers(HttpMethod.DELETE, "/books/**").hasRole("ADMIN")
						.and()
						.csrf().disable()
						.formLogin().disable();
			}
			
		}	
		
		
##  The Entry Point

-	In Spring MVC application security will enabled by default if we try to access the resources
-	the authentication process may automatically trigger when an un-authenticated client tries to access a secured resource
-	This process usually redirects to a login page so that the user can enter credentials
-	However, for a REST Web Service,this behaviour doesn’t make much sense
-	We should be able to authenticate only by a request to the correct URI and if the user is not authenticated all requests should simply fail with a 401 UNAUTHORIZED status code
-	Spring Security handles this automatic triggering of the authentication process with the concept of an Entry Point
-	Keeping in mind that this functionality doesn’t make sense in the context of the REST Service, we define the new custom entry point to simply return 401 when triggered:


			@Component
			public class RestAuthenticationEntryPoint implements AuthenticationEntryPoint {
				
				@Override
				public void commence(HttpServletRequest request, HttpServletResponse response, 
					AuthenticationException authException) throws IOException {
					 
					response.sendError(HttpServletResponse.SC_UNAUTHORIZED, 
					  "Unauthorized");
				}
			}
			
			
			
			
			
			
			
			
##	The Login Form for REST

-	Login form in RESt API doesn't make any sense and we should disable that
-	Using


			http
			.httpBasic()
			.and()
			.authorizeRequests()
			.loginForm().disable()
			
##	Authentication should Return 200 Instead of 301


-	By default, form login will answer a successful authentication request with a 301 MOVED PERMANENTLY status code; 
	-	this makes sense in the context of an actual login form which needs to redirect after login.
	
-	However, for a RESTful web service, the desired response for a successful authentication should be 200 OK.
-	We do this by injecting a custom authentication success handler in the form login filter, to replace the default one

	
			public class MySavedRequestAwareAuthenticationSuccessHandler 
			  extends SimpleUrlAuthenticationSuccessHandler {
			 
				private RequestCache requestCache = new HttpSessionRequestCache();
			 
				@Override
				public void onAuthenticationSuccess(
				  HttpServletRequest request,
				  HttpServletResponse response, 
				  Authentication authentication) 
				  throws ServletException, IOException {
			  
					SavedRequest savedRequest
					  = requestCache.getRequest(request, response);
			 
					if (savedRequest == null) {
						clearAuthenticationAttributes(request);
						return;
					}
					String targetUrlParam = getTargetUrlParameter();
					if (isAlwaysUseDefaultTargetUrl()
					  || (targetUrlParam != null
					  && StringUtils.hasText(request.getParameter(targetUrlParam)))) {
						requestCache.removeRequest(request, response);
						clearAuthenticationAttributes(request);
						return;
					}
			 
					clearAuthenticationAttributes(request);
				}
			 
				public void setRequestCache(RequestCache requestCache) {
					this.requestCache = requestCache;
				}
			}

