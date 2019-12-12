# Login-et-Logout
connexion, deconnexion


# model.py :

      class Profile(models.Model):
          user = models.OneToOneField(User, on_delete=models.CASCADE, related_name = "profileUser")
          # Initialisation a la creation
          image = models.ImageField(upload_to="profile", blank=True)
          email = models.EmailField(max_length=254)
          contact = models.CharField(max_length=250)

          @receiver(post_save, sender=User)
          def create_user_profile(sender, instance, created, **kwargs):
              if created:
                  Profile.objects.create(user=instance)

          @receiver(post_save, sender=User)
          def save_user_profile(sender, instance, created, **kwargs):
              instance.profileUser.save() ## prend le related_name qui le lie au model User
              
# views.py : 

      def register(request):
          if request.method == "POST" :
              success = False
              username = request.POST.get('username')
              nom = request.POST.get('nom')
              prenom = request.POST.get('prenom')
              email = request.POST.get('email')
              image = request.FILES.get('image')
              contact = request.POST.get('contact')
              password = request.POST.get('password')
              repassword = request.POST.get('repassword')
              print(nom, prenom, email, contact)

              if password == repassword : 
                  user = User(
                      username=username,
                      first_name=prenom,
                      last_name=nom,
                      email=email,
                  )
                  try :
                      user.save()
                      print(user)
                      user.profileUser.image=image
                      user.profileUser.contact=contact
                      user.profileUser.email=email
                      user.profileUser.save()
                      user.password=password
                      user.set_password(user.password)
                      user.save()
                      # prof = Profile(nom=nom, prenom=prenom, email=email, image=image, contact=contact)
                      # prof.save()
                      print('************************* succes ***********************************')
                      success = True
                      response = "votre inscription a ete effectuée avec succes"
                      return redirect('connexion')
                  except:
                      success = False
                      response = " error, veillez verifier votre connexion "

          return render(request, 'pages/register.html')



      def connexion(request):
          if request.method == 'POST':
              username = request.POST.get('username')
              password = request.POST.get('password')
              print('******************************************************', username, password )
              user = authenticate(username=username, password=password)
              if user is not None and user.is_active :
                  login(request, user)
                  return redirect('home')
              else:
                  return redirect('connexion')
          return render(request, 'pages/login.html')


      def deconnexion(request):
          logout(request)
          return redirect(reverse(connexion))

