VPN-
====
this is code for create vpn connection using c# .
just asking you IPAddress or phone number and password

Just using dotRas api, u can download ditras api then referenced dll in your project.
this code is worked for windows xp,windows 7 ,windows 8,as well as server environment.

      
        public const string ConnectionName = "Test"
        private RasHandle handle = null;
        RasEntry entry;
      public  bool IsExistCon = false;
       
        private void BtnDial_Click(object sender, EventArgs e)
        {
            foreach (RasConnection rasCon in RasConnection.GetActiveConnections())
            {
                if (rasCon.EntryName == ConnectionName)
                {
                    IsExistCon = true;
                }
               
            }
            if (IsExistCon == true)
            {

                BtnDial.Enabled = false;
                BtnHangUp.Enabled = true;
                StatusTextBox.AppendText("VPN Connection already running");
            }
            else
            {

                //string Rpath= System.Windows.Forms.Application.StartupPath+"\\Resources\\rasphone.pbk";
                //string ppath = Directory.GetParent(Application.ExecutablePath).FullName+"\\Pbk\\";
                string targetPath1 = Path.Combine(
                   Environment.GetFolderPath(
                       Environment.SpecialFolder.ApplicationData)) + "\\Microsoft\\Network\\Connections\\Pbk";
                string targetPath = Path.Combine(
                       Environment.GetFolderPath(
                           Environment.SpecialFolder.ApplicationData)) + "\\Microsoft\\Network\\Connections\\Pbk";
                bool checkUser = false;
                if (targetPath.Contains("Roaming"))
                {
                    rasUsersPhoneBook.Open(targetPath + "\\rasphone.pbk");
                    checkUser = true;
                }
                else
                {
                    rasUsersPhoneBook.Open();

                }
               
                DateTime srt = DateTime.Now;
              
                string VPName = ConnectionName;
               

                entry = RasEntry.CreateVpnEntry(VPName, txtIPaddress.Text, RasVpnStrategy.Default,
                    RasDevice.GetDeviceByName("(PPTP)", RasDeviceType.Vpn));
                entry.Options.RemoteDefaultGateway = false;
                entry.Options.Internet = true;

                if (!this.rasUsersPhoneBook.Entries.Contains(entry.Name))
                {
                    this.rasUsersPhoneBook.Entries.Add(entry);
                }

                this.StatusTextBox.Clear();
                this.Dialer.EntryName = ConnectionName;
                if (checkUser == true)
                {
                    this.Dialer.PhoneBookPath = targetPath + "\\rasphone.pbk";
                }
                else
                {
                    this.Dialer.PhoneBookPath = RasPhoneBook.GetPhoneBookPath(RasPhoneBookType.AllUsers);
                }
                try
                {
                    // Set the credentials the dialer should use.
                    this.Dialer.Credentials = new NetworkCredential(txtLoginID.Text, txtPassword.Text);

                    // NOTE: The entry MUST be in the phone book before the connection can be dialed.
                    // Begin dialing the connection; this will raise events from the dialer instance.
                    this.handle = this.Dialer.DialAsync();
                    //log.Info("Successfully Connected");
                    // Enable the disconnect button for use later.
                    this.BtnHangUp.Enabled = true;
                    this.BtnDial.Enabled = false;
                }
                catch (Exception ex)
                {
                    this.StatusTextBox.AppendText(ex.ToString());
                }
                DateTime endtime = DateTime.Now;
                TimeSpan avg = endtime - srt;
                lbname.Text = "Connected Time";
                lbConTime.Text = avg.TotalMilliseconds.ToString() + "ms";
            }
        }

        private void Dialer_StateChanged(object sender, StateChangedEventArgs e)
        {
            this.StatusTextBox.AppendText(e.State.ToString() + "\r\n");
        }

        private void Dialer_DialCompleted(object sender, DialCompletedEventArgs e)
        {   
            if (e.Cancelled)
            {
                this.StatusTextBox.AppendText("Cancelled!");
            }
            else if (e.TimedOut)
            {
                this.StatusTextBox.AppendText("Connection attempt timed out!");
            }
            else if (e.Error != null)
            {
                this.StatusTextBox.AppendText(e.Error.ToString());
            }
            else if (e.Connected)
            {
                this.StatusTextBox.AppendText("Connection successful!");
            }

            if (!e.Connected)
            {
                // The connection was not connected, disable the disconnect button.
                this.BtnHangUp.Enabled = false;
                this.StatusTextBox.AppendText("Connection successful Disconnected!");
            }

        }

        private void BtnHangUp_Click(object sender, EventArgs e)
        {
            Disconnect();
        }
        public void Disconnect()
        {
            
            DateTime srt = DateTime.Now;
            if (this.Dialer.IsBusy)
            {
                // The connection attempt has not been completed, cancel the attempt.
                this.Dialer.DialAsyncCancel();
            }
            else
            {
                // The connection attempt has completed, attempt to find the connection in the active connections.
                RasConnection connection = RasConnection.GetActiveConnectionByHandle(this.handle);
                //foreach (RasConnection connection in RasConnection.GetActiveConnections())
                //{
                    if (connection != null)
                    {
                        // The connection has been found, disconnect it.
                        if (connection.EntryName == ConnectionName)
                        {
                            connection.HangUp();
                            this.rasUsersPhoneBook.Entries.Remove(connection.EntryName);
                            rasUsersPhoneBook.Dispose();
                        }
                    }

                //}
            }
            this.StatusTextBox.Clear();
            this.StatusTextBox.AppendText("Connection successful Disconnected!");
            this.BtnHangUp.Enabled = false;
            this.BtnDial.Enabled = true;
            DateTime endtime = DateTime.Now;
            TimeSpan avg = endtime - srt;
            lbname.Text = "Disconnected Time";
            lbConTime.Text = avg.TotalMilliseconds.ToString()+"ms";
        
        }

       

        private void MainVPNWin_FormClosing(object sender, FormClosingEventArgs e)
        {
            Disconnect();
        }
          
