def main():
  crt.Session.LogUsingSessionOptions()
  crt.Screen.Send("OPENING a connection to: " + crt.GetScriptTab().Caption + "\n\r", True)

main()