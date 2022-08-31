# flask-note-7-useraccount-profile
用户如何修改post文和头像


    <div class="content-section">
        <div class="media">
        <img class="rounded-circle account-img" src="{{ image_file }}"> 改图像路径
        <div class="media-body">
            <h2 class="account-heading">{{ current_user.username}}</h2> 改成用户登陆名
            <p class="text-secondary">{{ current_user.email}}</p> 改成用户邮箱
        </div>
        </div>

1. 自动使用初始头像

        1.1 routes.py
  
          def account ():
            image_file = url_for('static', filename='profile_pics/' + current_user.image_file)
            return render_template('account.html', title='Account', image_file=image_file)
            加入了默认图像，之前models里面定义了profile就是用了默认图片且放在了profile里面。所以可以直接引用。


2. 更新邮箱和用户名
        
        2.1 更新用户名和邮箱的区域
            forms.py 需要更新一个新的表格：form
            在上面要引入file相关文件
            
            class UpdateAccountForm(FlaskForm):    和注册表格类似，但是具体项目要自己更新
                username = StringField("Username", validators=[DataRequired(), Length(min=2, max=20)])
                email = StringField("Email", validators=[DataRequired(), Email()])
                picture = FileField('Update Profile Picture', validators=[FileAllowed(['jpg','png'])])
                submit = SubmitField("Update")

                def validate_username(self, username): 
                    if username.data != current_user.username: 新输入的数据不等于当前，则用当前的作为新的username
                        user = User.query.filter_by(username=username.data).first() 
                        if user:     保证输入的新用户不要和已存在其他用户重合。注意：一旦更新，原有数据就被清除了，所以不会和自己重复
                            raise ValidationError('That username is taken. Please choose a different one.')

                def validate_email(self, email): 和上面一样
                    if email.data != current_user.email:
                        user = User.query.filter_by(email=email.data).first()
                        if user:
                            raise ValidationError('That email is taken. Please choose a different one.')

        2.2 登陆才有的界面，修改在 account.html
            建立新《div》，和register的界面差不多，所以可以copy，但是没有密码和确认密码部分。因而删除了一些。
            需要增加的部分是尾部的图片提交
            。。。
                    <div class="form-group">
                        {{ form.picture.label() }}  标题
                        <br>
                        {{ form.picture(class="form-control-label") }} 控制键-传文件的按钮子
                        <br>
                        {% if form.picture.errors %}    功能：如果出问题，出现红色的警戒提醒"text-danger"
                            {% for error in form.picture.errors %}
                                <span class="text-danger">{{ error }}</span>
                            {% endfor %}
                        {% endif %}
                    </div>
                    
        2.3 routes.py 包含了图片和用户名、邮箱
            在这个路径项下，存数据。成功则flash一个绿色标识，而且重新导回自己。
        
            @app.route("/account", methods=["GET", "POST"])
            @login_required
            def account ():
                form = UpdateAccountForm()
                if form.validate_on_submit():
                    if form.picture.data:
                        picture_file = save_picture(form.picture.data)
                        current_user.image_file = picture_file
                    current_user.username = form.username.data
                    current_user.email = form.email.data
                    db.session.commit()
                    flash('Your account has been updated', 'success')
                    return redirect(url_for('account'))
                elif request.method == 'GET':           或者使用get，即是让新更新的用户名和邮箱存入
                    form.username.data = current_user.username
                    form.email.data = current_user.email
                image_file = url_for('static', filename='profile_pics/' + current_user.image_file)
                return render_template('account.html', title='Account', image_file=image_file, form=form)


3. 修改profile图像
        
        3.1 加入图像的标识和类型 —— 见上一步的2.2
        
        3.2 储存图像到指定位置
            
            def save_picture(form_picture):
                random_hex = secrets.token_hex(8)       -用随意的数字组合来编码照片
                _, f_ext = os.path.splitext(form_picture.filename )     下划线表示的是picture_fn 
                picture_fn = random_hex + f_ext         新名字 = 随机数字+文件名
                picture_path = os.path.join(app.root_path, 'static/profile_pics', picture_fn) 路径把它们合起来

                output_size = (125,125)  这里是改变图片大小，任意新图片文件都转化成125 X 125。可以省内存。后面会存一大一小两个图片
                i = Image.open(form_picture) 
                i.thumbnail(output_size) 缓存缩略图
                i.save(picture_path) 存在路径中

                return picture_fn  返回储存图片文件
        
        3.3 改变图片大小，省内存



