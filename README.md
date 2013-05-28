//Main Window
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using System.Threading.Tasks;
using System.Windows;
using System.Windows.Controls;
using System.Windows.Data;
using System.Windows.Documents;
using System.Windows.Input;
using System.Windows.Media;
using System.Windows.Media.Imaging;
using System.Windows.Navigation;
using System.Windows.Shapes;
using System.Data.SqlClient;
using System.Data;
using Rozrahunkova.Properties;

namespace Rozrahunkova
{
    /// <summary>
    /// Логика взаимодействия для MainWindow.xaml
    /// </summary>
    public partial class MainWindow : Window
    {
        public MainWindow()
        {
            InitializeComponent();
        }

        SqlConnection SC = new SqlConnection(@"Data Source=ALEX\SQLEXPRESS;Initial Catalog=NULP_BRO;Integrated Security=True;Connect Timeout=15;Encrypt=False;TrustServerCertificate=False");
        SqlDataAdapter da = new SqlDataAdapter();

        string state = (string)Settings.Default["Lg"];

        private void Check()
        {
            if (state == "0")
            {
                SetUkr();
            }
            else if (state == "1")
            {
                SetEng();
            }
        }

        private void WindowLoaded(object sender, RoutedEventArgs e)
        {
            if (state == "0")
            {
                ukr.IsChecked = true;
                eng.IsChecked = false;
            }
            else if (state == "1")
            {
                ukr.IsChecked = false;
                eng.IsChecked = true;
            }
            Check();
            //MessageBox.Show(SC.State.ToString());
        }

        private void select(string ss)
        {
            DataTable ds = new DataTable();
            da.SelectCommand = new SqlCommand(ss, SC);
            da.Fill(ds);
            DG.ItemsSource = ds.DefaultView;
        }

        private void ShowButtonClick(object sender, RoutedEventArgs e)
        {
            //викликаємо функцію, для того щоб задала розміри датагрідів, залежності від вибраного бо деколи треба звільнити 1 датагрід
            Window_SizeChanged(sender, null);

            //за замовчуванням робимо 1 датагрід видимим а ліст вю невидимим
            //а потім взалежності від вибору будуть відображатися
            DG.Visibility = System.Windows.Visibility.Visible;
            LV.Visibility = System.Windows.Visibility.Collapsed;

            SC.Open();
            ComboBoxItem cbi = (ComboBoxItem)Combo.SelectedItem;
            if (cbi.Content.ToString() == "Курси" || cbi.Content.ToString() == "Courses")
            {
                select("SELECT subject_t.Id , subject_t.Name_subject, course_t.Startdate, course_t.Enddate, teacher_t.Name_teacher, teacher_t.Surname_Teacher, course_t.Status FROM [NULP_BRO].[Subject] subject_t RIGHT JOIN [NULP_BRO].[Course] course_t ON  subject_t.Id = course_t.Id_course RIGHT JOIN [NULP_BRO].[Teacher] teacher_t ON course_t.Id_course = teacher_t.Id");
                StartCourseButton.Visibility = System.Windows.Visibility.Visible;
                StopCourseButton.Visibility = System.Windows.Visibility.Visible;
                //робимо видимим 2 датагрід шоб відображати підписаних студентів
                DG2.Visibility = System.Windows.Visibility.Visible;
                SC.Close();
                return;
            }
            else if (cbi.Content.ToString() == "Предмети" || cbi.Content.ToString() == "Subjects")
            {
                DG.Visibility = System.Windows.Visibility.Collapsed;
                LV.Visibility = System.Windows.Visibility.Visible;
                ShowSubjects();
            }
            else if (cbi.Content.ToString() == "Викладачі" || cbi.Content.ToString() == "Teachers")
                select("SELECT Name_teacher, Surname_Teacher, Name_subject FROM [NULP_BRO].[Teacher] right join NULP_BRO.Subject on NULP_BRO.Teacher.Id = NULP_BRO.Subject.Id");

            else if (cbi.Content.ToString() == "Студенти" || cbi.Content.ToString() == "Students")
                select("SELECT * FROM [NULP_BRO].[Student]");  // тут тре вписати шоб воно відбражало на якій курси підписаний,

            SC.Close();
            //MessageBox.Show("done");
            //робимо невидимими
            StartCourseButton.Visibility = System.Windows.Visibility.Collapsed;
            StopCourseButton.Visibility = System.Windows.Visibility.Collapsed;
            DG2.Visibility = System.Windows.Visibility.Collapsed;
        }

        private void AddCourse(object sender, RoutedEventArgs e)
        {
            Window addcourse = new AddCourse();
            addcourse.Show();
        }

        private void AddTeacher(object sender, RoutedEventArgs e)
        {
            Addteacher addt = new Addteacher();
            addt.Show();
        }

        private void Exitbtn(object sender, RoutedEventArgs e)
        {
            this.Close();
        }

        private void AddSubject(object sender, RoutedEventArgs e)
        {
            Addsubjects adds = new Addsubjects();
            adds.Show();
        }

        private void DG_SelectionChanged(object sender, SelectionChangedEventArgs e)
        {
            //коли клікаємо на предметі показує хто на нього підписаний
        }

        private void StartCourseButton_Click(object sender, RoutedEventArgs e)
        {
            //якшо курсу невибрано то непідписуємся (бо нема на шо)
            if (DG.SelectedItem != null)
            {
                //дістаєм Id курсу, для того щоб зчитати з стик таблиці кількість записаних студентів
                //та кількість буде дорівнювати кількості однакових ідішок курсу
                //яких ми вибираємо нижче
                DataRowView row = (DataRowView)DG.SelectedItems[0];
                string id_course = row[0].ToString(); //дістаємо 1 колонку з ряду тобто id


                //дістаєм всі записи про вибраний нами курс(id_course) з стик. таблиці
                //ми шукаємо колонку Id_course, там де це поле = id_course(тому на який ми клікнули)
                SqlCommand GetIdCommand = new SqlCommand("SELECT Id_course FROM dbo.[Student-Course] WHERE Id_course = '" + id_course + "'", SC);
                SC.Open();
                SqlDataReader IDreader = GetIdCommand.ExecuteReader();

                int NumberOfID = 0;
                //поки воно читає ми рахуємо
                while (IDreader.Read())
                {
                    //рахуємо кількість ідішок, для нашого курсу
                    NumberOfID++;
                }
                SC.Close();
                IDreader.Close();

                //якщо кількість ідішок(NumberOfID) більша рівна 10 то миможемо стартувати курс
                if (NumberOfID >= 10)
                {
                    //задаємо курс як такий що читається
                    UpdateCourseStatus("Читається");
                }
                else
                {
                    string localstate = (string)Settings.Default["Lg"];
                    if (localstate == "0")
                    {
                        MessageBox.Show("На курс підписалося недостатньо студентів, щоб його стартувати");
                    }
                    else if (localstate == "1")
                    {
                        MessageBox.Show("This course has not enough students to get started");
                    }
                }
            }
            else
            {
                string localstate = (string)Settings.Default["Lg"];
                if (localstate == "0")
                {
                    MessageBox.Show("Виберіть курс!");
                }
                else if (localstate == "1")
                {
                    MessageBox.Show("Select course!");
                }
            }
            //оновлюємо датагрід
            ShowButtonClick(sender, e);
        }

        private void StopCourseButton_Click(object sender, RoutedEventArgs e)
        {
            //задаємо курс як Завершений
            UpdateCourseStatus("Завершений");
            //оновлюємо датагрід
            ShowButtonClick(sender, e);
        }

        private void UpdateCourseStatus(string Status)
        {
            //якшо курсу невибрано то нестуртуєм (бо нема на шо)
            if (DG.SelectedItem != null)
            {
                //дістаєм Id курсу шоб потім загнати уполе статус
                DataRowView row = (DataRowView)DG.SelectedItems[0];
                int id_course = (int)row[0]; //дістаємо 1 колонку з ряду тобто id

                try
                {
                    SC.Open();
                    da.UpdateCommand = new SqlCommand("UPDATE NULP_BRO.Course SET Status = @Status WHERE Id_course = @ID", SC);
                    da.UpdateCommand.Parameters.Add("@Status", SqlDbType.VarChar).Value = Status;
                    da.UpdateCommand.Parameters.Add("@ID", SqlDbType.Int).Value = id_course;

                    da.UpdateCommand.ExecuteNonQuery();
                    SC.Close();
                    //MessageBox.Show("Готово");
                }
                catch (Exception ex)
                {
                    //MessageBox.Show(ex.Message);
                    SC.Close();
                }
            }
            else
            {
                string localstate = (string)Settings.Default["Lg"];
                if (localstate == "1")
                {
                    MessageBox.Show("Select course!");
                }
                else if (localstate == "0")
                {
                    MessageBox.Show("Виберіть курс!");
                }
            }
        }

        //коли клікаємо на курс в 1 датагріді то в 2 дата гріді відображаються студенти шо підписані на той курс
        private void ShowStudent(object sender, SelectionChangedEventArgs e)
        {
            ComboBoxItem cbi = (ComboBoxItem)Combo.SelectedItem;
            if (DG.SelectedItem != null && cbi.Content.ToString() == "Курси" || cbi.Content.ToString() == "Courses")
            {
                try
                {
                    DataRowView row = (DataRowView)DG.SelectedItems[0];
                    string id_course = row[0].ToString(); //дістаємо 1 колонку з ряду тобто id
                    //дістаєм id студента
                    SqlCommand GetIDCommand = new SqlCommand("SELECT Id_student FROM [NULP_BRO].[Student] WHERE Name_student='" + App.Current.Properties["name"] + "' AND Surname = '" + App.Current.Properties["surname"] + "'", SC);
                    SC.Open();
                    SqlDataReader Namereader = GetIDCommand.ExecuteReader();

                    int id_student = 0;
                    while (Namereader.Read()) //заганяємо ід в лист ідішок
                    {
                        id_student = Namereader.GetInt32(0);

                    }
                    Namereader.Close();

                    DataTable dt = new DataTable();
                    SqlDataAdapter da = new SqlDataAdapter();
                    da.SelectCommand = new SqlCommand("select  s.Name_student , s.Surname from [NULP_BRO].[Course] as c inner join [Student-Course] as sc on c.Id_course = sc.Id_course inner join [NULP_BRO].[Student] s on s.Id_student = sc.Id_student where sc.Id_course ='" + id_course + "'", SC);

                    da.Fill(dt);
                    DG2.ItemsSource = dt.DefaultView;

                    SC.Close();
                    //MessageBox.Show("Готово");
                }
                catch (Exception ex)
                {
                    SC.Close();
                }
            }
        }

        private void Window_SizeChanged(object sender, SizeChangedEventArgs e)
        {
            //дістаємо вільний простір, тобто все що лишається  на формі після кобто і кнопок + трохи вільного місця
            double freeSpace = this.ActualHeight - 140;
            //присвоюємо 2 датагрідам однакові розміри шо = вільному поділеному на 2
            ComboBoxItem cbi = (ComboBoxItem)Combo.SelectedItem;
            if (cbi.Content.ToString() == "Курси" || cbi.Content.ToString() == "Courses")
            {
                DG.Height = freeSpace / 2;
                DG2.Height = freeSpace / 2;
            }
            else if (cbi.Content.ToString() == "Предмети" || cbi.Content.ToString() == "Subjects")
            {
                LV.Height = freeSpace;
            }
            else
            {
                DG.Height = freeSpace;
            }
        }
        private void SetUkr()
        {
            ShowButton.Content = "Показати";
            ComboCourses.Content = "Курси";
            ComboSubjects.Content = "Предмети";
            Teachers.Content = "Викладачі";
            ComboStudent.Content = "Студенти";
            StartCourseButton.Content = "Стартувати курс";
            StopCourseButton.Content = "Завершити курс";
            FileItem.Header = "Файл";
            LangSubItem.Header = "Мова";
            LogoutItem.Header = "Вийти";
            ExitSubItem.Header = "Вихід";
            AddItem.Header = "Додати";
            AboutItem.Header = "Про програму";
            TeacherItem.Header = "Додати викладача";
            SubjectAdd.Header = "Додати предмет";
            CourseAdd.Header = "Додати курс";

            SubjectColumn.Header = "Предмет";
            ModuleColumn.Header = "Модулі";
            TopicColumn.Header = "Теми";
            LectureColumn.Header = "Лекції";
            PractColumn.Header = "Практичні";
            LabColumn.Header = "Лабораторні";

        }
        private void SetEng()
        {
            ShowButton.Content = "Show";
            ComboCourses.Content = "Courses";
            ComboSubjects.Content = "Subjects";
            Teachers.Content = "Teachers";
            ComboStudent.Content = "Students";
            StartCourseButton.Content = "Start course";
            StopCourseButton.Content = "Stop course";
            FileItem.Header = "File";
            LangSubItem.Header = "Language";
            LogoutItem.Header = "Log out";
            ExitSubItem.Header = "Exit";
            AddItem.Header = "Add";
            AboutItem.Header = "About";
            TeacherItem.Header = "Add teacher";
            SubjectAdd.Header = "Add subject";
            CourseAdd.Header = "Add course";

            SubjectColumn.Header = "Subject";
            ModuleColumn.Header = "Modules";
            TopicColumn.Header = "Topics";
            LectureColumn.Header = "Lectures";
            PractColumn.Header = "Practicals";
            LabColumn.Header = "Labs";
        }

        private void SetUkr(object sender, RoutedEventArgs e)
        {
            Settings.Default["Lg"] = "0";
            Settings.Default.Save();
            ukr.IsChecked = true;
            eng.IsChecked = false;
            SetUkr();

        }

        private void SetEng(object sender, RoutedEventArgs e)
        {
            Settings.Default["Lg"] = "1";
            Settings.Default.Save();
            ukr.IsChecked = false;
            eng.IsChecked = true;
            SetEng();
        }

        private void ShowAboutWindow(object sender, RoutedEventArgs e)
        {
            AboutWindow aw = new AboutWindow();
            aw.Show();
        }


        private void ShowSubjects()
        {
            LV.Items.Clear();
            SqlCommand GetSubjectNameCommand = new SqlCommand("SELECT Id, Name_subject FROM [NULP_BRO].[Subject] order by NULP_BRO.Subject.Id", SC);
            SqlDataReader SubjectNamereader = GetSubjectNameCommand.ExecuteReader();
            //поки воно читає ми рахуємо

            List<string> ListOfSubject = new List<string>();
            List<int> ListOfSubjectId = new List<int>();
            while (SubjectNamereader.Read())
            {
                //тут є список предметів
                ListOfSubject.Add(SubjectNamereader.GetString(1));
                ListOfSubjectId.Add(SubjectNamereader.GetInt32(0));
            }
            SubjectNamereader.Close();


            for (int i = 0; i < ListOfSubjectId.Count; i++)
            {
                //для кожного предмету ми шукаємо його модуль і записуємо в 
                SqlCommand GetModuleCommand = new SqlCommand("SELECT Name_module FROM [NULP_BRO].Module where NULP_BRO.Module.Id = '" + ListOfSubjectId[i] + "'", SC);
                SqlDataReader Modulereader = GetModuleCommand.ExecuteReader();
                //поки воно читає ми рахуємо

                List<int> ListOfModuleid = new List<int>();
                string Modules = "";
                while (Modulereader.Read())
                {
                    Modules += Modulereader.GetString(0) + "\n";
                }
                Modulereader.Close();

                List<int> ListOfTopicId = new List<int>();
                string Topic = "";
                //вибираємо ім"я теми де ListOfModuleid[j]= Id з таблиці NULP_BRO.Module
                SqlCommand GetTopicCommand = new SqlCommand("SELECT Name_topic, Id FROM [NULP_BRO].Topic where NULP_BRO.Topic.Id = '" + ListOfSubjectId[i] + "'", SC);
                SqlDataReader Topicreader = GetTopicCommand.ExecuteReader();
                //поки воно читає ми рахуємо

                while (Topicreader.Read())
                {
                    Topic += Topicreader.GetString(0) + "\n";
                }
                Topicreader.Close();

                string Lecture = "";
                //вибираємо ім"я теми де ListOfModuleid[j]= Id з таблиці NULP_BRO.Module
                SqlCommand GetLectureCommand = new SqlCommand("SELECT Name_lecture FROM [NULP_BRO].Lecture where NULP_BRO.Lecture.Id = '" + ListOfSubjectId[i] + "'", SC);
                SqlDataReader Lecturereader = GetLectureCommand.ExecuteReader();
                //поки воно читає ми рахуємо

                while (Lecturereader.Read())
                {
                    Lecture += Lecturereader.GetString(0) + "\n";
                    //ListOfTopicId.Add(Lecturereader.GetInt32(1)); //дістаємо айдішки тем для даного предмету модуля
                }
                Lecturereader.Close();

                string Lab = "";
                //вибираємо ім"я теми де ListOfModuleid[j]= Id з таблиці NULP_BRO.Module
                SqlCommand GetLabCommand = new SqlCommand("SELECT Name_laba FROM [NULP_BRO].Laba where NULP_BRO.Laba.Id = '" + ListOfSubjectId[i] + "'", SC);
                SqlDataReader Labreader = GetLabCommand.ExecuteReader();
                //поки воно читає ми рахуємо

                while (Labreader.Read())
                {
                    Lab += Labreader.GetString(0) + "\n";
                    //ListOfTopicId.Add(Lecturereader.GetInt32(1)); //дістаємо айдішки тем для даного предмету модуля
                }
                Labreader.Close();


                string Pract = "";
                //вибираємо ім"я теми де ListOfModuleid[j]= Id з таблиці NULP_BRO.Module
                SqlCommand GetPractCommand = new SqlCommand("SELECT Name_practical FROM [NULP_BRO].Practical where NULP_BRO.Practical.Id = '" + ListOfSubjectId[i] + "'", SC);
                SqlDataReader Practreader = GetPractCommand.ExecuteReader();
                //поки воно читає ми рахуємо

                while (Practreader.Read())
                {
                    Pract += Practreader.GetString(0) + "\n";
                    //ListOfTopicId.Add(Lecturereader.GetInt32(1)); //дістаємо айдішки тем для даного предмету модуля
                }
                Practreader.Close();

                //додаємо срані елементи в lv
                LV.Items.Add(new { SubjectName = ListOfSubject[i], ModuleName = Modules, TopicName = Topic, LectureName = Lecture, PractName = Pract, LabName = Lab });
            }
        }

        private void LogOutItem_Click(object sender, RoutedEventArgs e)
        {
            Window v = new StartWindow();
            v.Show();
            Close();
        }
    }
}
//Start Window

using System;
using System.Collections.Generic;
using System.Data;
using System.Data.SqlClient;
using System.Linq;
using System.Text;
using System.Threading.Tasks;
using System.Windows;
using System.Windows.Controls;
using System.Windows.Data;
using System.Windows.Documents;
using System.Windows.Input;
using System.Windows.Media;
using System.Windows.Media.Imaging;
using System.Windows.Shapes;
using Rozrahunkova.Properties;
namespace Rozrahunkova
{
    /// <summary>
    /// Логика взаимодействия для StartWindow.xaml
    /// </summary>
    public partial class StartWindow : Window
    {
        public StartWindow()
        {
            InitializeComponent();
        }

        string state = (string)Settings.Default["Lg"];

        private void Check()
        {
            if(state == "0")
            {
                SetUkr();
            }
            else if(state == "1")
            {
                SetEng();
            }
        }

        private void UpdateCombo(object sender, SelectionChangedEventArgs e)
        {
            //при виборі категорії юзера клієнта, ми задаємо параметри вікна,
            //тобто видимість політ кнопок і тп....
            ComboBoxItem cbi = (ComboBoxItem)StartCombo.SelectedItem;
            if (cbi.Content.ToString() == "Адміністратор" || cbi.Content.ToString() == "Administrator")
            {
                PassLabel.Visibility = System.Windows.Visibility.Visible;
                PassTextBox.Visibility = System.Windows.Visibility.Visible;
                NameTextBox.Visibility = System.Windows.Visibility.Collapsed;
                SurNameTextBox.Visibility = System.Windows.Visibility.Collapsed;
                NameLabel.Visibility = System.Windows.Visibility.Collapsed;
                SurNameLabel.Visibility = System.Windows.Visibility.Collapsed;
            }
            else if (cbi.Content.ToString() == "Студент" || cbi.Content.ToString() == "Student")
            {
                PassTextBox.Visibility = System.Windows.Visibility.Visible;
                NameTextBox.Visibility = System.Windows.Visibility.Visible;
                SurNameTextBox.Visibility = System.Windows.Visibility.Visible;
                NameLabel.Visibility = System.Windows.Visibility.Visible;
                SurNameLabel.Visibility = System.Windows.Visibility.Visible;
                PassLabel.Visibility = System.Windows.Visibility.Visible;
                this.Height = 280;
            }

            enterButton.IsEnabled = true;
        }

        private void Enter(object sender, RoutedEventArgs e)
        {
            //при виборі юзера задаємо кнопці відкрити відповідне вікно
            ComboBoxItem cbi = (ComboBoxItem)StartCombo.SelectedItem;
            if (cbi.Content.ToString() == "Адміністратор" || cbi.Content.ToString() == "Administrator")
            {
                if (PassTextBox.Password == "123")
                {
                    Window v = new MainWindow();
                    v.Show();
                    Close();
                }
                else
                {
                    if (state == "0")
                        MessageBox.Show("Пароль невірний");
                    else if (state == "1")
                        MessageBox.Show("Password is wrong");
                }
            }
            else if (cbi.Content.ToString() == "Студент" || cbi.Content.ToString() == "Student")
            {
                if (NameTextBox.Text == "" || SurNameTextBox.Text == "" || PassTextBox.Password == "")
                {
                    if(state =="0")
                        MessageBox.Show("Введіть ім'я, прізвище та пароль!");
                    else if(state =="1")
                        MessageBox.Show("Enter name, surname and password!");
                }
                else
                {
                    //перевіряємо чи введене імя студента і пароль є в списку і
                    CheckExistingOfStudent();
                }
            }
        }

        private void RegisterButton_Click(object sender, RoutedEventArgs e)
        {
            Window register = new Sign();
            register.Show();
        }

        SqlConnection SC = new SqlConnection(@"Data Source=ALEX\SQLEXPRESS;Initial Catalog=NULP_BRO;Integrated Security=True;Connect Timeout=15;Encrypt=False;TrustServerCertificate=False");
        SqlDataAdapter da = new SqlDataAdapter();

        private void CheckExistingOfStudent()
        {
            
            SC.Open();

            //дістаємо лист імен
            SqlCommand GetNameCommand = new SqlCommand("SELECT Name_student, Surname, Password FROM [NULP_BRO].[Student]", SC);
            SqlDataReader Namereader = GetNameCommand.ExecuteReader();
            List<string> NameList = new List<string>();
            List<string> SurNameList = new List<string>();

            List<string> PassList = new List<string>();
            while (Namereader.Read())
            {
                NameList.Add(Namereader.GetString(0));
                SurNameList.Add(Namereader.GetString(1));
                PassList.Add(Namereader.GetString(2));
            }

            SC.Close();

            //в циклі порівнюємо чи введене ім'я і прізвище відповідають тим шо ми дістали вище
           
            for (int i = 0; i < NameList.Count; i++)
            {
                if (NameList[i] == NameTextBox.Text && SurNameList[i]==SurNameTextBox.Text && PassList[i] == PassTextBox.Password.ToString())
                {
                    App.Current.Properties["name"] = NameTextBox.Text;              //так я передаю глобальні змінні між вікнами
                    App.Current.Properties["surname"] = SurNameTextBox.Text;       //в програмі
                    Window v = new UserWindow();
                    v.Show();
                    Close();
                    return;
                }
            }
            if (state == "0")
                MessageBox.Show("Ви ввели неправильно ім'я, призвіще або пароль");
             else if (state == "1")
                MessageBox.Show("Incorrect name, surname or password");
        }

        private void TextBox_KeyDown(object sender, KeyEventArgs e)
        {
            if (e.Key == Key.Enter)
                Enter(sender, e);
        }

        private void SetUkr()
        {
            CategoryText.Text = "Виберіть категорію:";
            enterButton.Content = "Вхід";
            RegisterButton.Content = "Зареєструватись";
            ComboAmin.Content = "Адміністратор";
            ComboStudent.Content = "Студент";
            NameLabel.Text = "Введіть ім'я:";
            SurNameLabel.Text = "Введіть призвіще:";
            PassLabel.Text = "Пароль:";
        }
        private void SetEng()
        {

            CategoryText.Text = "Select category:";
            enterButton.Content = "Log in";
            RegisterButton.Content = "Sign up";
            ComboAmin.Content = "Administrator";
            ComboStudent.Content = "Student";
            NameLabel.Text = "Enter name:";
            SurNameLabel.Text = "Enter surname:";
            PassLabel.Text = "Password:";
        }

        private void Window_Loaded_1(object sender, RoutedEventArgs e)
        {
            Check();
        }
    }
}
//User Window
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using System.Threading.Tasks;
using System.Windows;
using System.Windows.Controls;
using System.Windows.Data;
using System.Windows.Documents;
using System.Windows.Input;
using System.Windows.Media;
using System.Windows.Media.Imaging;
using System.Windows.Shapes;
using System.Data.SqlClient;
using System.Data;
using Rozrahunkova.Properties;
namespace Rozrahunkova
{
    /// <summary>
    /// Логика взаимодействия для UserWindow.xaml
    /// </summary>
    public partial class UserWindow : Window
    {
        public UserWindow()
        {
            InitializeComponent();
        }

        string state = (string)Settings.Default["Lg"];

        private void Check()
        {
            if (state == "0")
            {
                SetUkr();
            }
            else if (state == "1")
            {
                SetEng();
            }
        }

        DataTable ds = new DataTable();
        SqlConnection SC = new SqlConnection(@"Data Source=ALEX\SQLEXPRESS;Initial Catalog=NULP_BRO;Integrated Security=True;Connect Timeout=15;Encrypt=False;TrustServerCertificate=False");
        SqlDataAdapter da = new SqlDataAdapter();

        private void UserWindowLoaded(object sender, RoutedEventArgs e)
        {
            if (state == "0")
            {
                ukr.IsChecked = true;
                eng.IsChecked = false;
            }
            else if (state == "1")
            {
                ukr.IsChecked = false;
                eng.IsChecked = true;
            }
            Check();
        }


        private void select(string ss)
        {
            da.SelectCommand = new SqlCommand(ss, SC);
            ds.Clear();
            da.Fill(ds);
            DG.ItemsSource = ds.DefaultView;
        }

        private void ShowButtonClick(object sender, RoutedEventArgs e)
        {
            ComboBoxItem cbi = (ComboBoxItem)Combo.SelectedItem;
            if (cbi.Content.ToString() == "Курси" || cbi.Content.ToString() == "Courses")
            {
                SC.Close();
                SC.Open();
                select("SELECT subject_t.Id , subject_t.Name_subject, course_t.Startdate, course_t.Enddate, teacher_t.Name_teacher, teacher_t.Surname_Teacher, course_t.Status FROM [NULP_BRO].[Subject] subject_t RIGHT JOIN [NULP_BRO].[Course] course_t ON  subject_t.Id = course_t.Id_course RIGHT JOIN [NULP_BRO].[Teacher] teacher_t ON course_t.Id_course = teacher_t.Id");
                SignButton.Visibility = System.Windows.Visibility.Visible;
                UnSignButton.Visibility = System.Windows.Visibility.Collapsed;
                SC.Close();
            }
            else if (cbi.Content.ToString() == "Вибрані курси" || cbi.Content.ToString() == "Selected courses")
            {
                UnSignButton.Visibility = System.Windows.Visibility.Visible;
                SignButton.Visibility = System.Windows.Visibility.Collapsed; 
                ShowSelectedCourses();
            }
        }

        private void SignButton_Click(object sender, RoutedEventArgs e)
        {
            CourseSing();
        }

        private void UnSignButton_Click(object sender, RoutedEventArgs e)
        {
            SC.Close();

            if (DG.SelectedItem != null)
            {                
                //дістаєм Id курсу шоб потім видалити зі стик. табл.
                DataRowView row = (DataRowView)DG.SelectedItems[0];
                string name_course = row[0].ToString(); //дістаємо 1 колонку з ряду тобто name

                //
                SC.Open();
                SqlCommand getid = new SqlCommand("SELECT Id FROM [NULP_BRO].[Subject] WHERE Name_subject='" + name_course + "'", SC);
                SqlDataReader Idreader = getid.ExecuteReader();
                int id_course = 0;
                while (Idreader.Read())
                {
                    id_course = Idreader.GetInt32(0);
                }
                SC.Close();
                Idreader.Dispose();

                // дістаєм Id студента 

                SqlCommand GetIDCommand = new SqlCommand("SELECT Id_student FROM [NULP_BRO].[Student] WHERE Name_student='" + App.Current.Properties["name"] + "' AND Surname = '" + App.Current.Properties["surname"] + "'", SC);
                SC.Open();
                SqlDataReader Namereader = GetIDCommand.ExecuteReader();

                int id_student = 0;
                while (Namereader.Read()) //заганяємо ід в лист ідішок
                {
                    id_student = Namereader.GetInt32(0);
                }

                SC.Close();
                Namereader.Close();

                //Видаляєм зі  стик. табл.
                try
                {
                    SC.Open();
                    string query = String.Format("delete from [dbo].[Student-Course] where Id_student = '" + id_student + "' and Id_course = '" + id_course + "'");
                    SqlCommand signstudent = new SqlCommand(query, SC);// записуєм студента
                    signstudent.ExecuteNonQuery();
                    signstudent.Dispose();
                    SC.Close();
                    if (state == "0")
                    {
                        MessageBox.Show("Готово");
                    }
                    else if (state == "1")
                    {
                        MessageBox.Show("Done");
                    }                    
                    ShowSelectedCourses();
                }
                catch
                {

                }
            }
            else
            {
                if (state == "0")
                {

                    MessageBox.Show("Виберіть курс!");
                }
                else if (state == "1")
                {
                    MessageBox.Show("Select course!");
                }
            }
        }

        private void CourseSing()
        {
            DataRowView row = (DataRowView)DG.SelectedItems[0];
            string id_course = row[0].ToString(); //дістаємо 1 колонку з ряду тобто id

            if (row[6].ToString().Contains("Проводиться набір") || row[6].ToString().Contains("Читається"))
            {
                //дістаєм id студента
                SqlCommand GetIDCommand = new SqlCommand("SELECT Id_student FROM [NULP_BRO].[Student] WHERE Name_student='" + App.Current.Properties["name"] + "' AND Surname = '" + App.Current.Properties["surname"] + "'", SC);
                SC.Open();
                SqlDataReader Namereader = GetIDCommand.ExecuteReader();

                int id_student = 0;
                while (Namereader.Read()) //заганяємо ід в лист ідішок
                {
                    id_student = Namereader.GetInt32(0);
                }

                SC.Close();
                Namereader.Close();
                //записуєм у стик. табл.
                try
                {
                    SC.Close();
                    SC.Open();
                    string query = String.Format("INSERT INTO [dbo].[Student-Course](Id_student, Id_course) VALUES('" + id_student + "','" + id_course + "')");
                    SqlCommand signstudent = new SqlCommand(query, SC);// записуєм студента
                    signstudent.ExecuteNonQuery();
                    signstudent.Dispose();
                    SC.Close();
                    if (state == "0")
                    {
                        MessageBox.Show("Готово");
                    }
                    else if (state == "1")
                    {
                        MessageBox.Show("Done");
                    }
                }
                catch (Exception ex)
                {
                    if (state == "0")
                    {
                        MessageBox.Show("Ви вже підписані на цей курс");
                    }
                    else if (state == "1")
                    {
                        MessageBox.Show("You already subscribed to this course");
                    }
                    SC.Close();
                }
            }
            else if (row[6].ToString().Contains("Завершений"))
            {
                if (state == "0")
                    MessageBox.Show("Ви не можете записатися на завершений курс!");
                else
                    MessageBox.Show("You can`n subscribe on finished course!");
            }            
        }


        private void ShowSelectedCourses()
        {
            SC.Close();
            //дістаєм id студента
            SqlCommand GetIDCommand = new SqlCommand("SELECT id_student FROM [NULP_BRO].[Student] WHERE Name_student='" + App.Current.Properties["name"] + "' AND Surname = '" + App.Current.Properties["surname"] + "'", SC);
            SC.Open();
            SqlDataReader Namereader = GetIDCommand.ExecuteReader();

            int id_student = 0;
            while (Namereader.Read()) //заганяємо ід в лист ідішок
            {
                id_student = Namereader.GetInt32(0);
            }
            SC.Close();
            Namereader.Close();

            try
            {
                SC.Open();

                DataTable dt = new DataTable();
                SqlDataAdapter da = new SqlDataAdapter();
                da.SelectCommand = new SqlCommand("select sb.Name_subject , c.Startdate , c.Enddate   , t.Name_teacher , t.Surname_Teacher , c.Status from [NULP_BRO].[Subject] sb inner join [NULP_BRO].[Course] c on sb.Id = c.Id_course inner join [NULP_BRO].[Teacher] t on t.Id = c.Id_course inner join [Student-Course] as sc on sc.Id_course = c.Id_course inner join [NULP_BRO].[Student] as st on st.id_student = sc.Id_student where sc.Id_student = '"+id_student+"'", SC);
                da.Fill(dt);
                DG.ItemsSource = dt.DefaultView;
                
                SC.Close();
            }
            catch (Exception ex)
            {
                MessageBox.Show(ex.Message);
            }
        }

        private void SetUkr()
        {
            LoginText.Text = "Ви ввійшли як: " + App.Current.Properties["name"] + " " + App.Current.Properties["surname"];
            ShowButton.Content = "Показати";
            SignButton.Content = "Підписатись";
            FileItem.Header = "Файл";
            LangSubItem.Header = "Мова";
            LogOutItem.Header = "Вийти";
            ExitSubItem.Header = "Вихід";
            AboutItem.Header = "Про програму";
            UnSignButton.Content = "Відписатись";
            ComboCourse.Content = "Курси";
            ComboSelectedCourses.Content="Вибрані курси";
        }

        private void SetEng()
        {
            LoginText.Text = "You logged in as: " + App.Current.Properties["name"] + " " + App.Current.Properties["surname"];
            FileItem.Header = "File";
            LangSubItem.Header = "Language";
            LogOutItem.Header = "Log out";
            ExitSubItem.Header = "Exit";
            AboutItem.Header = "About...";
            ShowButton.Content = "Show";
            SignButton.Content = "Subscribe";
            UnSignButton.Content = "Unsubscribe";
            ComboCourse.Content = "Courses";
            ComboSelectedCourses.Content = "Selected courses";
        }

        private void LogOut_Click(object sender, RoutedEventArgs e)
        {            
            Window V = new StartWindow();
            V.Show();
            Close();
        }

        private void Exitbtn(object sender, RoutedEventArgs e)
        {
            Close();
        }

        private void SetEng(object sender, RoutedEventArgs e)
        {
            Settings.Default["Lg"] = "1";
            Settings.Default.Save();
            ukr.IsChecked = false;
            eng.IsChecked = true;
            SetEng();
        }

        private void SetUkr(object sender, RoutedEventArgs e)
        {
            Settings.Default["Lg"] = "0";
            Settings.Default.Save();
            ukr.IsChecked = true;
            eng.IsChecked = false;
            SetUkr();
        }

        private void ShowAboutWindow_Click(object sender, RoutedEventArgs e)
        {
            AboutWindow aw = new AboutWindow();
            aw.Show();
        }
    }
}
//Register Window

using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using System.Threading.Tasks;
using System.Windows;
using System.Windows.Controls;
using System.Windows.Data;
using System.Windows.Documents;
using System.Windows.Input;
using System.Windows.Media;
using System.Windows.Media.Imaging;
using System.Windows.Shapes;
using System.Data.SqlClient;
using Rozrahunkova.Properties;

namespace Rozrahunkova
{
    /// <summary>
    /// Логика взаимодействия для Sign.xaml
    /// </summary>
    public partial class Sign : Window
    {
        public Sign()
        {
            InitializeComponent();
        }

        SqlConnection con = new SqlConnection(@"Data Source=ALEX\SQLEXPRESS;Initial Catalog=NULP_BRO;Integrated Security=True;Connect Timeout=15;Encrypt=False;TrustServerCertificate=False");
        SqlCommand cmd = new SqlCommand();

        string state = (string)Settings.Default["Lg"];

        private void Check()
        {
            if (state == "0")
            {
                SetUkr();
            }
            else if (state == "1")
            {
                SetEng();
            }
        }

        private void CancelButton_Click(object sender, RoutedEventArgs e)
        {
            Close();
        }

        private void RegisterButton_Click(object sender, RoutedEventArgs e)
        {
            if (NameFeed.Text != "" && SurNameFeed.Text != "")
            {
                try
                {
                    string query = String.Format("INSERT INTO [NULP_BRO].[Student](Name_student, Surname, Password) VALUES('" + NameFeed.Text + "','" + SurNameFeed.Text + "','" + PassFeed.Text + "')");
                    cmd = new SqlCommand(query, con);
                    con.Open();
                    cmd.ExecuteNonQuery();
                    cmd.Dispose();
                    con.Close();
                    if (state == "0")
                    {
                        MessageBox.Show("Готово");
                    }
                    else if (state == "1")
                    {
                        MessageBox.Show("Done");
                    }
                    Close();
                }
                catch (Exception ex)
                {
                    MessageBox.Show(ex.Message);
                }
            }
            else
            {
                if (state == "0")
                {
                    MessageBox.Show("Не правильний ввід");
                }
                else if (state == "1")
                {
                    MessageBox.Show("Wrong typing");
                }
            }
            
        }

        private void TextBox_KeyDown(object sender, KeyEventArgs e)
        {
            if (e.Key == Key.Enter)
                RegisterButton_Click(sender, e);
        }
        private void SetUkr()
        {
            NameLabel.Content = "Ваше ім'я:";
            SurnameLabel.Content = "Ваше призвіще:";
            PassLabel.Content = "Пароль:";
            RegisterButton.Content = "Зареєструватись";
            CancelButton.Content = "Відміна";
        }
        private void SetEng()
        {
            NameLabel.Content = "Your name:";
            SurnameLabel.Content = "Your surname:";
            PassLabel.Content = "Password:";
            RegisterButton.Content = "Sign up";
            CancelButton.Content = "Cancel";
           
        }

        private void Window_Loaded_1(object sender, RoutedEventArgs e)
        {
            Check();
        }
    }
}
//Add Teacher Window

using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using System.Threading.Tasks;
using System.Windows;
using System.Windows.Controls;
using System.Windows.Data;
using System.Windows.Documents;
using System.Windows.Input;
using System.Windows.Media;
using System.Windows.Media.Imaging;
using System.Windows.Shapes;
using System.Data.SqlClient;
using System.Data;
using Rozrahunkova.Properties;


namespace Rozrahunkova
{
    /// <summary>
    /// Логика взаимодействия для Addteacher.xaml
    /// </summary>
    public partial class Addteacher : Window
    {
        public Addteacher()
        {
            InitializeComponent();
        }
        SqlCommand cmd = new SqlCommand();
        SqlCommand sc = new SqlCommand();
        SqlConnection con = new SqlConnection(@"Data Source=ALEX\SQLEXPRESS;Initial Catalog=NULP_BRO;Integrated Security=True;Connect Timeout=15;Encrypt=False;TrustServerCertificate=False");

        SqlDataAdapter da = new SqlDataAdapter();

        private void Addbtn(object sender, RoutedEventArgs e)
        {
            if (Nametxt.Text != "" && Surnametxt.Text != "")
            {
                try
                {
                    con.Open();
                    da.InsertCommand = new SqlCommand("INSERT INTO [NULP_BRO].[Teacher](Name_teacher,Surname_teacher)VALUES(@Nametxt,@Surnametxt)", con);
                    da.InsertCommand.Parameters.Add("@Nametxt", SqlDbType.NChar).Value = Nametxt.Text;
                    da.InsertCommand.Parameters.Add("@Surnametxt", SqlDbType.NChar).Value = Surnametxt.Text;
                    da.InsertCommand.ExecuteNonQuery();
                    con.Close();
                    //MessageBox.Show("Готово");
                }
                catch (Exception ex)
                {
                    MessageBox.Show(ex.Message);
                    con.Close();
                }

            }
            Close();
        }

        private void CloseButtonClick(object sender, RoutedEventArgs e)
        {
            Close();
        }
        string state = (string)Settings.Default["Lg"];

        private void Check()
        {
            if (state == "0")
            {
                SetUkr();
            }
            else if (state == "1")
            {
                SetEng();
            }
        }

        private void Window_Loaded(object sender, RoutedEventArgs e)
        {
            Check();
        }

        private void SetUkr()
        {
            Label1.Content = "Ім`я викладача:";
            Label2.Content = "Прізвище викладача:";
            AddButton.Content = "Додати";
            ExitButton.Content = "Відміна";
        }

        private void SetEng()
        {
            Label1.Content = "Name of teacher:";
            Label2.Content = "Surname of teacher:";
            AddButton.Content = "Add teacher";
            ExitButton.Content = "Cancel";
        }
    }
}
//Add Subject Window

using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using System.Threading.Tasks;
using System.Windows;
using System.Windows.Controls;
using System.Windows.Data;
using System.Windows.Documents;
using System.Windows.Input;
using System.Windows.Media;
using System.Windows.Media.Imaging;
using System.Windows.Shapes;
using System.Data.SqlClient;
using System.Data;
using Rozrahunkova.Properties;

namespace Rozrahunkova
{
    /// <summary>
    /// Логика взаимодействия для Addsubjects.xaml
    /// </summary>
    public partial class Addsubjects : Window
    {
        SqlConnection con = new SqlConnection(@"Data Source=ALEX\SQLEXPRESS;Initial Catalog=NULP_BRO;Integrated Security=True;Connect Timeout=15;Encrypt=False;TrustServerCertificate=False");
        SqlCommand sc = new SqlCommand();
        SqlDataAdapter da = new SqlDataAdapter();

        public Addsubjects()
        {
            InitializeComponent();
        }
        
        string state = (string)Settings.Default["Lg"];

        private void Check()
        {
            if (state == "0")
            {
                SetUkr();
            }
            else if (state == "1")
            {
                SetEng();
            }
        }


        private void CancelButton_Click(object sender, RoutedEventArgs e)
        {
            Close();
        }

        private void AddSubjectButton_Click(object sender, RoutedEventArgs e)
        {
            string surname = (string)CourseCombo.SelectedValue;

            int id_teacher = 0;
            SqlCommand GetIDCommand = new SqlCommand("SELECT Id_course FROM [NULP_BRO].[Course] WHERE Name_course = '" + surname + "'", con);
            con.Open();
            SqlDataReader Namereader = GetIDCommand.ExecuteReader();
            while (Namereader.Read()) //заганяємо ід в лист ідішок
            {
                id_teacher = Namereader.GetInt32(0);
                //IdList.Add(Namereader.GetInt32(0));
            }
            con.Close();
            Namereader.Close();
            if (SubjectNameFeed.Text != "")
            {
                try
                {
                    con.Open();
                    da.InsertCommand = new SqlCommand("INSERT INTO [NULP_BRO].Subject(Id_course,Name_subject)VALUES(@Id_course,@SubjectNameFeed)", con);
                    da.InsertCommand.Parameters.Add("@Id_course", SqlDbType.Int).Value = id_teacher;
                    da.InsertCommand.Parameters.Add("@SubjectNameFeed", SqlDbType.NChar).Value = SubjectNameFeed.Text;
                    da.InsertCommand.ExecuteNonQuery();
                    con.Close();
                    //MessageBox.Show("Готово");
                }
                catch (Exception ex)
                {
                    MessageBox.Show(ex.Message);
                    con.Close();
                }
            }
        }

        private void AddModuleButton_Click(object sender, RoutedEventArgs e)
        {
            string surname = (string)SubjectCombo.SelectedValue;

            int id_teacher = 0;
            SqlCommand GetIDCommand = new SqlCommand("SELECT Id FROM [NULP_BRO].[Subject] WHERE Name_subject = '" + surname + "'", con);
            con.Open();
            SqlDataReader Namereader = GetIDCommand.ExecuteReader();
            while (Namereader.Read()) //заганяємо ід в лист ідішок
            {
                id_teacher = Namereader.GetInt32(0);
                //IdList.Add(Namereader.GetInt32(0));
            }
            con.Close();
            Namereader.Close();
            if (ModuletNameFeed.Text != "")
            {
                try
                {
                    con.Open();
                    da.InsertCommand = new SqlCommand("INSERT INTO [NULP_BRO].Module(Name_module,Id)VALUES(@ModuletNameFeed,@Id)", con);

                    da.InsertCommand.Parameters.Add("@ModuletNameFeed", SqlDbType.NChar).Value = ModuletNameFeed.Text;
                    da.InsertCommand.Parameters.Add("@Id", SqlDbType.Int).Value = id_teacher;
                    da.InsertCommand.ExecuteNonQuery();
                    con.Close();
                    //MessageBox.Show("Готово");
                }
                catch (Exception ex)
                {
                    MessageBox.Show(ex.Message);
                    con.Close();
                }
            }
        }

        private void AddPracticalButton_Click(object sender, RoutedEventArgs e)
        {
            string surname = (string)Topic1Combo.SelectedValue;

            int id_teacher = 0;
            SqlCommand GetIDCommand = new SqlCommand("SELECT Id FROM [NULP_BRO].[Subject] WHERE Name_subject = '" + surname + "'", con);
            con.Open();
            SqlDataReader Namereader = GetIDCommand.ExecuteReader();
            while (Namereader.Read()) //заганяємо ід в лист ідішок
            {
                id_teacher = Namereader.GetInt32(0);
                //IdList.Add(Namereader.GetInt32(0));
            }
            con.Close();
            Namereader.Close();
            if (PracticalNameFeed.Text != "")
            {
                try
                {
                    con.Open();
                    da.InsertCommand = new SqlCommand("INSERT INTO [NULP_BRO].Practical(Id,Name_practical)VALUES(@Id,@PracticalNameFeed)", con);
                    da.InsertCommand.Parameters.Add("@Id", SqlDbType.Int).Value = id_teacher;
                    da.InsertCommand.Parameters.Add("@PracticalNameFeed", SqlDbType.NChar).Value = PracticalNameFeed.Text;

                    da.InsertCommand.ExecuteNonQuery();
                    con.Close();
                    //MessageBox.Show("Готово");
                }
                catch (Exception ex)
                {
                    MessageBox.Show(ex.Message);
                    con.Close();
                }
            }
        }

        private void AddLabButton_Click(object sender, RoutedEventArgs e)
        {
            string surname = (string)Topic2Combo.SelectedValue;

            int id_teacher = 0;
            SqlCommand GetIDCommand = new SqlCommand("SELECT Id FROM [NULP_BRO].[Subject] WHERE Name_subject = '" + surname + "'", con);
            con.Open();
            SqlDataReader Namereader = GetIDCommand.ExecuteReader();
            while (Namereader.Read()) //заганяємо ід в лист ідішок
            {
                id_teacher = Namereader.GetInt32(0);
                //IdList.Add(Namereader.GetInt32(0));
            }
            con.Close();
            Namereader.Close();
            if (LabNameFeed.Text != "")
            {
                try
                {
                    con.Open();
                    da.InsertCommand = new SqlCommand("INSERT INTO [NULP_BRO].Laba(Id,Name_laba)VALUES(@Id,@LabNameFeed)", con);
                    da.InsertCommand.Parameters.Add("@Id", SqlDbType.Int).Value = id_teacher;
                    da.InsertCommand.Parameters.Add("@LabNameFeed", SqlDbType.NChar).Value = LabNameFeed.Text;

                    da.InsertCommand.ExecuteNonQuery();
                    con.Close();
                    //MessageBox.Show("Готово");
                }
                catch (Exception ex)
                {
                    MessageBox.Show(ex.Message);
                    con.Close();
                }
            }
        }

        private void AddLectureButton_Click(object sender, RoutedEventArgs e)
        {
            string surname = (string)TopicCombo.SelectedValue;

            int id_teacher = 0;
            SqlCommand GetIDCommand = new SqlCommand("SELECT Id FROM [NULP_BRO].[Subject] WHERE Name_subject = '" + surname + "'", con);
            con.Open();
            SqlDataReader Namereader = GetIDCommand.ExecuteReader();
            while (Namereader.Read()) //заганяємо ід в лист ідішок
            {
                id_teacher = Namereader.GetInt32(0);
                //IdList.Add(Namereader.GetInt32(0));
            }
            con.Close();
            Namereader.Close();
            if (LectureNameFeed.Text != "")
            {
                try
                {
                    con.Open();
                    da.InsertCommand = new SqlCommand("INSERT INTO [NULP_BRO].Lecture(Id,Name_lecture)VALUES(@Id,@LectureNameFeed)", con);
                    da.InsertCommand.Parameters.Add("@Id", SqlDbType.Int).Value = id_teacher;
                    da.InsertCommand.Parameters.Add("@LectureNameFeed", SqlDbType.NChar).Value = LectureNameFeed.Text;

                    da.InsertCommand.ExecuteNonQuery();
                    con.Close();
                    //MessageBox.Show("Готово");
                }
                catch (Exception ex)
                {
                    MessageBox.Show(ex.Message);
                    con.Close();
                }

            }
        }

        #region заганяємо дані в комбобокси
        void Combo1()
        {
            // SqlDataReader dr = new SqlDataReader();
            string Query = "SELECT * FROM [NULP_BRO].[Course]";
            sc = new SqlCommand(Query, con);
            try
            {
                con.Open();
                SqlDataReader dr = sc.ExecuteReader();
                while (dr.Read())
                {
                    string Sname = dr.GetString(dr.GetOrdinal("Name_course"));
                    CourseCombo.Items.Add(Sname);
                }
            }
            catch (Exception ex)
            {

                // MessageBox.Show();
            } con.Close();
        }
        void Combo2()
        {
            // SqlDataReader dr = new SqlDataReader();
            string Query = "SELECT * FROM  [NULP_BRO].Subject";
            sc = new SqlCommand(Query, con);
            try
            {
                con.Open();
                SqlDataReader dr = sc.ExecuteReader();
                while (dr.Read())
                {
                    string Sname = dr.GetString(dr.GetOrdinal("Name_subject"));
                    SubjectCombo.Items.Add(Sname);
                }
            }
            catch (Exception ex)
            {

                // MessageBox.Show();
            } con.Close();
        }
        void Combo3()
        {
            // SqlDataReader dr = new SqlDataReader();
            string Query = "SELECT * FROM  [NULP_BRO].Subject";
            sc = new SqlCommand(Query, con);
            try
            {
                con.Open();
                SqlDataReader dr = sc.ExecuteReader();
                while (dr.Read())
                {
                    string Sname = dr.GetString(dr.GetOrdinal("Name_subject"));
                    ModuleCombo.Items.Add(Sname);
                }
            }
            catch (Exception ex)
            {

                // MessageBox.Show();
            }
            con.Close();
        }
        void Combo4()
        {
            // SqlDataReader dr = new SqlDataReader();
            string Query = "SELECT * FROM  [NULP_BRO].Subject";
            sc = new SqlCommand(Query, con);
            try
            {
                con.Open();
                SqlDataReader dr = sc.ExecuteReader();
                while (dr.Read())
                {
                    string Sname = dr.GetString(dr.GetOrdinal("Name_subject"));
                    TopicCombo.Items.Add(Sname);
                }
            }
            catch (Exception ex)
            {

                // MessageBox.Show();
            } con.Close();
        }
        void Combo5()
        {
            // SqlDataReader dr = new SqlDataReader();
            string Query = "SELECT * FROM  [NULP_BRO].Subject";
            sc = new SqlCommand(Query, con);
            try
            {
                con.Open();
                SqlDataReader dr = sc.ExecuteReader();
                while (dr.Read())
                {
                    string Sname = dr.GetString(dr.GetOrdinal("Name_subject"));
                    Topic1Combo.Items.Add(Sname);
                }
            }
            catch (Exception ex)
            {

                // MessageBox.Show();
            } con.Close();
        }
        void Combo6()
        {
            // SqlDataReader dr = new SqlDataReader();
            string Query = "SELECT * FROM  [NULP_BRO].Subject";
            sc = new SqlCommand(Query, con);
            try
            {
                con.Open();
                SqlDataReader dr = sc.ExecuteReader();
                while (dr.Read())
                {
                    string Sname = dr.GetString(dr.GetOrdinal("Name_subject"));
                    Topic2Combo.Items.Add(Sname);
                }
            }
            catch (Exception ex)
            {

                // MessageBox.Show();
            } con.Close();
        }
        #endregion
        

        private void AddTopicButton_Click(object sender, RoutedEventArgs e)
        {
            string surname = (string)ModuleCombo.SelectedValue;

            int id_teacher = 0;
            SqlCommand GetIDCommand = new SqlCommand("SELECT Id FROM [NULP_BRO].[Subject] WHERE Name_subject = '" + surname + "'", con);
            con.Open();
            SqlDataReader Namereader = GetIDCommand.ExecuteReader();
            while (Namereader.Read()) //заганяємо ід в лист ідішок
            {
                id_teacher = Namereader.GetInt32(0);
                //IdList.Add(Namereader.GetInt32(0));
            }
            con.Close();
            Namereader.Close();
            if (TopicNameFeed.Text != "")
            {
                try
                {
                    con.Open();
                    da.InsertCommand = new SqlCommand("INSERT INTO [NULP_BRO].Topic(Name_topic, Id) VALUES(@TopicNameFeed, @Id)", con);

                    da.InsertCommand.Parameters.Add("@TopicNameFeed", SqlDbType.NChar).Value = TopicNameFeed.Text;
                    da.InsertCommand.Parameters.Add("@Id", SqlDbType.Int).Value = id_teacher;
                    da.InsertCommand.ExecuteNonQuery();
                    con.Close();
                    //MessageBox.Show("Готово");
                }
                catch (Exception ex)
                {
                    MessageBox.Show(ex.Message);
                    con.Close();
                }
            }
        }

        private void Combo_DropDownOpened(object sender, EventArgs e)
        {
            //при відкритті комбо оновлюємо контент
            CourseCombo.Items.Clear();
            ModuleCombo.Items.Clear();
            SubjectCombo.Items.Clear();
            TopicCombo.Items.Clear();
            Topic1Combo.Items.Clear();
            Topic2Combo.Items.Clear();

            Combo1();
            Combo2();
            Combo3();
            Combo4();
            Combo5();
            Combo6();
        }

        private void ExitButton_Click(object sender, RoutedEventArgs e)
        {
            Close();
        }

        private void Window_Loaded(object sender, RoutedEventArgs e)
        {
            Check();
        }

        private void SetUkr()
        {
            G1.Header = "Додавання предмету:";
            G2.Header = "Додавання модуля:";
            G3.Header = "Додавання теми:";
            G4.Header = "Додавання лекції:";
            G5.Header = "Додавання практичної:";
            G6.Header = "Додавання лабораторної:";
            Label1.Text = "Назва предмету:";
            Label2.Text = "Курс:";
            Label3.Text = "Назва модуля:";
            Label4.Text = "Належить до:";
            Label5.Text = "Назва теми:";
            Label6.Text = "Належить до:";
            Label7.Text = "Назва лекції:";
            Label8.Text = "Належить до:";
            Label9.Text = "Назва практичної:";
            Label10.Text = "Належить до:";
            Label11.Text = "Назва лабораторної:";
            Label12.Text = "Належить до:";
            AddSubjectButton.Content = "Додати предмет";
            AddModuleButton.Content = "Додати модуль";
            AddTopicButton.Content = "Додати тему";
            AddLectureButton.Content = "Додати лекцію";
            AddPracticalButton.Content = "Додати практичну";
            AddLabButton.Content = "Додати лабораторну";
            ExitButton.Content = "Вихід";
        }

        private void SetEng()
        {
            G1.Header = "Adding a subject:";
            G2.Header = "Adding a module:";
            G3.Header = "Adding a topic:";
            G4.Header = "Adding a lecture:";
            G5.Header = "Adding a practical:";
            G6.Header = "Adding a lab:";
            Label1.Text = "Name of subject:";
            Label2.Text = "Курс:";
            Label3.Text = "Name of module:";
            Label4.Text = "Belongs to:";
            Label5.Text = "Name of topic:";
            Label6.Text = "Belongs to:";
            Label7.Text = "Name of lecture:";
            Label8.Text = "Belongs to:";
            Label9.Text = "Name of practical:";
            Label10.Text = "Belongs to:";
            Label11.Text = "Name of lab:";
            Label12.Text = "Belongs to:";

            AddSubjectButton.Content = "Add subject";
            AddModuleButton.Content = "Add module";
            AddTopicButton.Content = "Add topic";
            AddLectureButton.Content = "Add lecture";
            AddPracticalButton.Content = "Add practical";
            AddLabButton.Content = "Add lab";
            ExitButton.Content = "Exit";
        }

    }
}
//Add Course Window

using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using System.Threading.Tasks;
using System.Windows;
using System.Windows.Controls;
using System.Windows.Data;
using System.Windows.Documents;
using System.Windows.Input;
using System.Windows.Media;
using System.Windows.Media.Imaging;
using System.Windows.Shapes;
using System.Data.SqlClient;
using System.Data;
using Rozrahunkova.Properties;

namespace Rozrahunkova
{
    /// <summary>
    /// Логика взаимодействия для AddCourse.xaml
    /// </summary>
    public partial class AddCourse : Window
    {
        public AddCourse()
        {
            InitializeComponent();
            Combo1();
        }

        SqlConnection con = new SqlConnection(@"Data Source=ALEX\SQLEXPRESS;Initial Catalog=NULP_BRO;Integrated Security=True;Connect Timeout=15;Encrypt=False;TrustServerCertificate=False");
        SqlCommand sc = new SqlCommand();
        SqlDataReader dr;
        string Id;

        private void AddСourseButton_Click(object sender, RoutedEventArgs e)
        {
            try
            {
                string surname = (string)TeacherCombo.SelectedValue;

                int id_teacher = 0;
                SqlCommand GetIDCommand = new SqlCommand("SELECT Id FROM [NULP_BRO].Teacher WHERE Surname_teacher = '" + surname + "'", con);
                con.Open();
                SqlDataReader Namereader = GetIDCommand.ExecuteReader();
                while (Namereader.Read()) //заганяємо ід в лист ідішок
                {
                    id_teacher = Namereader.GetInt32(0);
                }
                con.Close();
                Namereader.Close();

                DateTime data, data1;
                data = Convert.ToDateTime(StartFeed.Text);
                data1 = Convert.ToDateTime(EndFeed.Text);
                if (StartFeed.Text != "" && EndFeed.Text != "" && Namecourse.Text != "")
                {
                    try
                    {
                        string query1 = String.Format("INSERT INTO [NULP_BRO].[Course](Id_course, Startdate, Enddate, Name_course, Status) VALUES('" + id_teacher + "','" + data + "','" + data1 + "','" + Namecourse.Text + "','" + "Проводиться набір" + "')");
                        sc = new SqlCommand(query1, con);
                        con.Open();
                        sc.ExecuteNonQuery();
                        sc.Dispose();
                        con.Close();
                    }
                    catch (Exception ex)
                    {
                        MessageBox.Show(ex.Message);
                        con.Close();
                    }
                }
            }
            catch(Exception ex)
            {
                MessageBox.Show(ex.Message);
            }
            Close();
        }

        private void Combo1()
        {
            // SqlDataReader dr = new SqlDataReader();
            string Query = "SELECT * FROM [NULP_BRO].Teacher";
            sc = new SqlCommand(Query, con);
            try
            {
                con.Open();
                SqlDataReader dr = sc.ExecuteReader();
                while (dr.Read())
                {
                    TeacherCombo.Items.Add(dr.GetString(dr.GetOrdinal("Surname_teacher")));
                }
            }
            catch
            {
                // MessageBox.Show();
            }
            con.Close();
        }

        private void CancelButton_Click(object sender, RoutedEventArgs e)
        {
            Close();
        }

        string state = (string)Settings.Default["Lg"];

        private void Check()
        {
            if (state == "0")
            {
                SetUkr();
            }
            else if (state == "1")
            {
                SetEng();
            }
        }

        private void Window_Loaded(object sender, RoutedEventArgs e)
        {
            Check();
        }

        private void SetUkr()
        {
            Label1.Text = "Початок:";
            Label2.Text = "Закінчення:";
            Label3.Text = "Назва:";
            Label4.Content = "Викладач:";
            AddCourseButton.Content = "Додати курс";
            ExitButton.Content = "Відміна";
        }

        private void SetEng()
        {
            Label1.Text = "Start date:";
            Label2.Text = "End Date:";
            Label3.Text = "Name:";
            Label4.Content = "Teacher:";
            AddCourseButton.Content = "Add course";
            ExitButton.Content = "Cancel";
        }
    }
}
//About Window
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using System.Threading.Tasks;
using System.Windows;
using System.Windows.Controls;
using System.Windows.Data;
using System.Windows.Documents;
using System.Windows.Input;
using System.Windows.Media;
using System.Windows.Media.Imaging;
using System.Windows.Shapes;

namespace Rozrahunkova
{
    /// <summary>
    /// Логика взаимодействия для AboutWindow.xaml
    /// </summary>
    public partial class AboutWindow : Window
    {
        public AboutWindow()
        {
            InitializeComponent();
        }

        private void Button_Click_1(object sender, RoutedEventArgs e)
        {
            Close();
        }
    }
}
