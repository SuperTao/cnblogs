ini文件主要用于保存配置。之前一直以为是当作普通文本进行操作，读取里面的内容，再自己解析读取的文本。后来发现已经有写好的api函数：WritePrivateProfileString()和GetPrivateProfileString()。

然后查看怎么使用，直到看到有人已经封装成了类，直接使用即可。代码源自一下链接，稍微做了下笔记，记录于此。

参考链接：

　　http://blog.csdn.net/geekwangminli/article/details/7851673

　　http://www.cnblogs.com/zzyyll2/archive/2007/11/06/950584.html

IniFile.cs
```
using System;
using System.Collections.Generic;
using System.Text;
using System.Runtime.InteropServices;

namespace INIFILE
{
    public abstract class CustomIniFile
    {
        public CustomIniFile(string AFileName)
        {
            FFileName = AFileName;
        }
        private string FFileName;
        public string FileName
        {
            get { return FFileName; }
        }
        public virtual bool SectionExists(string Section)
        {
            List<string> vStrings = new List<string>();
            ReadSections(vStrings);
            return vStrings.Contains(Section);
        }
        public virtual bool ValueExists(string Section, string Ident)
        {
            List<string> vStrings = new List<string>();
            ReadSection(Section, vStrings);
            return vStrings.Contains(Ident);
        }
        public abstract string ReadString(string Section, string Ident, string Default);
        public abstract bool WriteString(string Section, string Ident, string Value);
        public abstract bool ReadSectionValues(string Section, List<string> Strings);
        public abstract bool ReadSection(string Section, List<string> Strings);
        public abstract bool ReadSections(List<string> Strings);
        public abstract bool EraseSection(string Section);
        public abstract bool DeleteKey(string Section, string Ident);
        public abstract bool UpdateFile();
    }
   /// <summary>
    /// 存储本地INI文件的类。
    /// </summary>
    public class IniFile : CustomIniFile
    {
        [DllImport("kernel32")]
        private static extern uint GetPrivateProfileString(
            string lpAppName,    // points to section name 
            string lpKeyName,    // points to key name 
            string lpDefault,    // points to default string 
            byte[] lpReturnedString,    // points to destination buffer 
            uint nSize,    // size of destination buffer 
            string lpFileName     // points to initialization filename 
        );

        [DllImport("kernel32")]
        private static extern bool WritePrivateProfileString(
            string lpAppName,    // pointer to section name 
            string lpKeyName,    // pointer to key name 
            string lpString,    // pointer to string to add 
            string lpFileName     // pointer to initialization filename 
        );

        /// <summary>
        /// 构造IniFile实例。
        /// <param name="AFileName">指定文件名</param>
        /// </summary>
        public IniFile(string AFileName)
            : base(AFileName)
        {
        }

        /// <summary>
        /// 析够IniFile实例。
        /// </summary>
        ~IniFile()
        {
            UpdateFile();
        }

        /// <summary>
        /// 读取字符串值。
        /// <param name="Ident">指定变量标识。</param>
        /// <param name="Section">指定所在区域。</param>
        /// <param name="Default">指定默认值。</param>
        /// <returns>返回读取的字符串。如果读取失败则返回该值。</returns>
        /// </summary>
        /* 读取key对应的value的值，如果对应的key不存在或者存在，但没有等号，就是用默认值Default */
        public override string ReadString(string Section, string Ident, string Default)
        {
            byte[] vBuffer = new byte[2048];
            uint vCount = GetPrivateProfileString(Section,
                Ident, Default, vBuffer, (uint)vBuffer.Length, FileName);
            return Encoding.Default.GetString(vBuffer, 0, (int)vCount);
        }
        /// <summary>
        /// 写入字符串值。
        /// </summary>
        /// <param name="Section">指定所在区域。</param>
        /// <param name="Ident">指定变量标识。</param>
        /// <param name="Value">所要写入的变量值。</param>
        /// <returns>返回写入是否成功。</returns>
        public override bool WriteString(string Section, string Ident, string Value)
        {
            return WritePrivateProfileString(Section, Ident, Value, FileName);
        }

        /// <summary>
        /// 获得区域的完整文本。(变量名=值格式)。
        /// </summary>
        /// <param name="Section">指定区域标识。</param>
        /// <param name="Strings">输出处理结果。</param>
        /// <returns>返回读取是否成功。</returns>
        public override bool ReadSectionValues(string Section, List<string> Strings)
        {
            List<string> vIdentList = new List<string>();
            // 先读取某个section中所有的key值
            if (!ReadSection(Section, vIdentList)) return false;
            // 根据key值获取value的值，并按照格式组成string
            foreach (string vIdent in vIdentList)
                Strings.Add(string.Format("{0}={1}", vIdent, ReadString(Section, vIdent, "")));
            return true;
        }

        /// <summary>
        /// 读取区域变量名列表。
        /// </summary>
        /// <param name="Section">指定区域名。</param>
        /// <param name="Strings">指定输出列表。</param>
        /// <returns>返回获取是否成功。</returns>
        /* 读取某一个section中所有的key的值*/
        public override bool ReadSection(string Section, List<string> Strings)
        {
            byte[] vBuffer = new byte[16384];
            //读取整个section中的key值，存到vBuffer中
            uint vLength = GetPrivateProfileString(Section, null, null, vBuffer,
                (uint)vBuffer.Length, FileName);
            int j = 0;
            for (int i = 0; i < vLength; i++)
                if (vBuffer[i] == 0)
                {
                    //将byte类型转成string,并保存到动态数组中, 按照0分开
                    //byte内容如下，每个key通过00分隔
                    //4E-61-6D-65-00-42-61-75-64-52-61-74-65-00-44-61-74-61-42-69-74-73-00-53-74-6F-70-42-69-74-73-00-47-5F-50-41-52-49-54-59
                    //将一长串字符组成string,就是将2个00之间的byte内容组成string,并添加到动态数组中
                    Strings.Add(Encoding.Default.GetString(vBuffer, j, i - j));
                    j = i + 1;
                }
            return true;
        }

        /// <summary>
        /// 读取区域名列表。
        /// </summary>
        /// <param name="Strings">指定输出列表。</param>
        /// <returns></returns>
        /* 读取的是中括号中的section名字 */
        public override bool ReadSections(List<string> Strings)
        {
            byte[] vBuffer = new byte[16384];
            uint vLength = GetPrivateProfileString(null, null, null, vBuffer,
                (uint)vBuffer.Length, FileName);
            int j = 0;
            for (int i = 0; i < vLength; i++)
                if (vBuffer[i] == 0)
                {
                    // 将数组中第j~i个byte的内容转成string,添加到list中
                    Strings.Add(Encoding.Default.GetString(vBuffer, j, i - j));
                    j = i + 1;
                }
            return true;
        }

        /// <summary>
        /// 删除指定区域。
        /// </summary>
        /// <param name="Section">指定区域名。</param>
        /// <returns>返回删除是否成功。</returns>
        /* 删除整个section的内容 */
        public override bool EraseSection(string Section)
        {
            return WritePrivateProfileString(Section, null, null, FileName);
        }

        /// <summary>
        /// 删除指定变量。
        /// </summary>
        /// <param name="Section">变量所在区域。</param>
        /// <param name="Ident">变量标识。</param>
        /// <returns>返回删除是否成功。</returns>
        /* 删除之后，key和value的值都不存在 */
        public override bool DeleteKey(string Section, string Ident)
        {
            return WritePrivateProfileString(Section, Ident, null, FileName);
        }

        /// <summary>
        /// 更新文件。
        /// </summary>
        /// <returns>返回更新是否成功。</returns>
        public override bool UpdateFile()
        {
            return WritePrivateProfileString(null, null, null, FileName);
        }
    }
}
```



Profile.cs
```
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;

namespace INIFILE
{
    class Profile
    {
        public static void LoadProfile()
        {
            //获取生成的可执行文件的目录
            string strPath = AppDomain.CurrentDomain.BaseDirectory;
            _file = new IniFile(strPath + "Cfg.ini");
            G_BAUDRATE = _file.ReadString("CONFIG", "BaudRate", "4800");    //读数据，下同
            G_DATABITS = _file.ReadString("CONFIG", "DataBits", "8");
            G_STOP = _file.ReadString("CONFIG", "StopBits", "1");
            G_PARITY = _file.ReadString("CONFIG", "Parity", "NONE");
        }

        public static void SaveProfile()
        {
            string strPath = AppDomain.CurrentDomain.BaseDirectory;
            _file = new IniFile(strPath + "Cfg.ini");
            _file.WriteString("CONFIG", "BaudRate", G_BAUDRATE);            //写数据，下同
            _file.WriteString("CONFIG", "DataBits", G_DATABITS);
            _file.WriteString("CONFIG", "StopBits", G_STOP);
            _file.WriteString("CONFIG", "G_PARITY", G_PARITY);
        }

        private static IniFile _file;//内置了一个对象

        public static string G_BAUDRATE = "1200";//给ini文件赋新值，并且影响界面下拉框的显示
        public static string G_DATABITS = "8";
        public static string G_STOP = "1";
        public static string G_PARITY = "NONE";
    }
}
```

Tony Liu

2016-9-26，Shenzhen