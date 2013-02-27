#Background
The configuration file of a program was modified but the output of a program was not changed as expected.
#Causes
The configuration file was one of **many** buffers opened by Emacs, and it was not saved after modification.
#Lessons learned
* Keep the workspace small, simple, and clean to avoid obvious mistakes.
* Similar mistakes that could be prevented:
  + Mistakenly editing a file with the same intended name but in a wrong folder
  + Linking the program with a static library that is not updated after code change