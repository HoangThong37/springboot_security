# SpringBoot-SpringSecurity
### Tạo database gồm 3 bảng

+ create table APP_USER <br>
(<br>
  USER_ID           BIGINT not null, <br>
  USER_NAME         VARCHAR(36) not null,<br>
  ENCRYTED_PASSWORD VARCHAR(128) not null,<br>
  ENABLED           BIT not null <br>
);<br>

+ alter table APP_USER<br>
  add constraint APP_USER_PK primary key (USER_ID);<br>

- alter table APP_USER<br>
  add constraint APP_USER_UK unique (USER_NAME);


+ Create table APP_ROLE<br>
(<br>
  ROLE_ID   BIGINT not null,<br>
  ROLE_NAME VARCHAR(30) not null<br>
);<br>

+ alter table APP_ROLE<br>
  add constraint APP_ROLE_PK primary key (ROLE_ID);<br>

+ alter table APP_ROLE<br>
  add constraint APP_ROLE_UK unique (ROLE_NAME);<br>

+ Create table USER_ROLE<br>
(<br>
  ID      BIGINT not null,<br>
  USER_ID BIGINT not null,<br>
  ROLE_ID BIGINT not null<br>
);<br>

+ alter table USER_ROLE<br>
  add constraint USER_ROLE_PK primary key (ID);<br>

+ alter table USER_ROLE<br>
  add constraint USER_ROLE_UK unique (USER_ID, ROLE_ID);<br>

+ alter table USER_ROLE<br>
  add constraint USER_ROLE_FK1 foreign key (USER_ID)<br>
  references APP_USER (USER_ID);<br>

+ alter table USER_ROLE<br>
  add constraint USER_ROLE_FK2 foreign key (ROLE_ID)<br>
  references APP_ROLE (ROLE_ID);<br>


#### Used by Spring Remember Me API.<br>
CREATE TABLE Persistent_Logins (

    username varchar(64) not null,
    series varchar(64) not null,
    token varchar(64) not null,
    last_used timestamp not null,
    PRIMARY KEY (series)

);

## Thêm dữ liệu vào DB
insert into App_User (USER_ID, USER_NAME, ENCRYTED_PASSWORD, ENABLED)values<br> 
(2, 'dbuser1', '$2a$10$PrI5Gk9L.tSZiW9FXhTS8O8Mz9E97k2FZbFvGFFaSsiTUIl.TCrFu', 1);<br>

insert into App_User (USER_ID, USER_NAME, ENCRYTED_PASSWORD, ENABLED)<br>
values (1, 'dbadmin1', '$2a$10$PrI5Gk9L.tSZiW9FXhTS8O8Mz9E97k2FZbFvGFFaSsiTUIl.TCrFu', 1);<br>

---

insert into app_role (ROLE_ID, ROLE_NAME)
values (1, 'ROLE_ADMIN');

insert into app_role (ROLE_ID, ROLE_NAME)
values (2, 'ROLE_USER');

---

insert into user_role (ID, USER_ID, ROLE_ID)
values (1, 1, 1);

insert into user_role (ID, USER_ID, ROLE_ID)
values (2, 1, 2);

insert into user_role (ID, USER_ID, ROLE_ID)
values (3, 2, 2);

## Nếu như muốn check xem password có đúng không thì mình viết 1 class riêng và kế thừa lại class AuthenticationProvider  nhu sau. Trong pthưc override authentication thì mình ktra password có đúng k ?

public class MyClass implements AuthenticationProvider {

public Authentication authenticate(Authentication authentication) throws AuthenticationException {
		if (authentication.getPrincipal() == null || authentication.getCredentials() == null) {
			throw new BadCredentialsException("Username or Password empty");
		}

		String username = authentication.getName();
        String password = authentication.getCredentials().toString();

		UserDetails userDetails = userDetailsService.loadUserByUsername(username);

		if (userDetails == null) {
			logger.info("User {} not existed", username);
			throw new BadCredentialsException("User not found");
		}

		// Authenticate
		String passwordHash = CommonUtils.md5(password);
		if(!passwordHash.equals(userDetails.getPassword())){
			logger.info("User {} has enter an invalid password", username);
			throw new BadCredentialsException("Invalid password");
			
		}

		UsernamePasswordAuthenticationToken result = new UsernamePasswordAuthenticationToken(userDetails,
				authentication.getCredentials(), userDetails.getAuthorities());
		
		logger.info("Leaving AirtimeAuthenticationProvider.authenticate()");

		return result;
	}public Authentication authenticate(Authentication authentication) throws AuthenticationException {

		logger.info("Enteriing AirtimeAuthenticationProvider.authenticate()");
		
		if (authentication.getPrincipal() == null || authentication.getCredentials() == null) {
			throw new BadCredentialsException("Username or Password empty");
		}

		
		String username = authentication.getName();
        String password = authentication.getCredentials().toString();

		UserDetails userDetails = userDetailsService.loadUserByUsername(username);

		if (userDetails == null) {
			logger.info("User {} not existed", username);
			throw new BadCredentialsException("User not found");
		}

		// Authenticate
		String passwordHash = CommonUtils.md5(password);
		if(!passwordHash.equals(userDetails.getPassword())){
			logger.info("User {} has enter an invalid password", username);
			throw new BadCredentialsException("Invalid password");
			
		}

		UsernamePasswordAuthenticationToken result = new UsernamePasswordAuthenticationToken(userDetails,
				authentication.getCredentials(), userDetails.getAuthorities());
		
		logger.info("Leaving AirtimeAuthenticationProvider.authenticate()");

		return result;
	}
  }


+ MỤC ĐÍCH BÀI HƯỚNG DẪN :
   - Spring security dùng làm ứng dụng phân quyền, bảo mật...  
    - Ở đây tùy thuộc vào user đăng nhập vào hệ thống là user hay admin mà ta cho phép họ vào trang web tương ứng. Ví dụ :
	+ Trang Home thì ai vào cũng được .
	+ Trang Admin thì chỉ có admin được vào và thấy được trang . Nếu là role user và vào trang Admin thì mình hiện thông báo lỗi bạn không có quyền.
	+ Trang User Info thì user và admin được phép vào. Cái này do mình hoàn toàn có thể thay đổi quyền trong database để phân quyền ai được phép vào trang nào.

2. Các khái niệm về Spring Security :
 * Authentication : Khi nói về authentication là ta nói về chức năng đăng nhập vào hệ thống. Authentication nghĩa là bạn có phải là người dùng của hệ thống hay không.
 
 * Authorization : Khi nói về authorization ta nói về quyền hạn được phép làm gì? Trong ví dụ trên mình có user và admin. Bước đầu tiên họ phải authentication. Xác thực mình là user trong hệ thống. Tiếp đến tuỳ vào role của mình là admin hay user mà mình chỉ có quyền truy cập một số trang nhất định thuộc thẩm quyền của mình.

3. Xây dựng ứng dụng Spring Security :
 + Luồng đi của ứng dụng mình như sau
       - User nhập vào username và password sau đó bấm login .
	   - Server sẽ nhận được request từ người dùng và chuyển tới controller tương ứng do ta cấu hình trong file configure của spring security .
	   - Controller sẽ gọi Service và Service sẽ gọi database để lấy thông tin authentication đúng không và role người dùng là gì?.<br>
		 Sau khi có thông tin đúng thì trả kết quả lại cho người dùng.

	   + Bước 1. Chuẩn bị database để lưu thông tin user và quyền
		        Mình dùng database để lưu thông tin người dùng và role (vai trò,được phép làm gì). Phục vụ cho việc truy vấn username và role có hợp lệ hay không.
				
		
	Để làm ứng dụng spring security mình sẽ lưu user name và quyền vào trong database. Ý nghĩa của từng bảng : 

		1. Table APP_USER dùng để lưu thông tin username và password. Khi người dùng đăng nhập họ truyền user name và password vào form sau đó code của mình sẽ query trong database xem là username và password có đúng như trong database không ?  Bước này chính là authentication.

		2. Table APP_ROLE dùng để xác định xem user sau khi login thành công thì được phép vào những trang nào . Ví dụ admin vào được 2 trang user và admin page . Nhưng user chỉ được phép vào 1 trang là user page.

		3. Table USER_ROLE là table dùng để nối 2 bảng APP_USER và APP_ROLE, nó được dùng để cho phép 1 user có thể có nhiều quyền. Ví dụ như admin có thể vào cả 2 trang user và admin .

+ Tạo database. 
+ Tạo cấu trúc database và tạo dữ liệu user và admin tại đây. Mọi người copy về và chạy script nhé : <br>
   -  https://github.com/HoangThong37/springboot_security.git

			Nếu chạy script xong thì mình sẽ có 2 users sau :
			username : dbuser1 - Password : 123
			username : dbadmin1 - Password : 123


	
