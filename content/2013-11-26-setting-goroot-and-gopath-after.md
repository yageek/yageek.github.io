---

title: "Setting GOROOT and GOPATH after installing Go with Homebrew"
date: 2013-11-26
comments: true
categories:
 - homebrew
 - go
---

<div class='post'>
I successfully installed <b>Go</b> by using the simplicity of Homebrew. Installation was fine but some errors occurs with doing a <b>go get </b>command or simply compiling the simple "hello world" program.<br /><br />After few seconds, I realized how I suck and forget to set up the <b>GOPATH</b> and <b>GOROOT</b> variables.<br /><br />With Homebrew, the base installation of <b>Go</b> rely under<b> /usr/local/Cellar/go/1.1.2</b>.<br /><br />Simply set <b>GOROOT</b> to <b>/usr/local/Cellar/go/1.1.2/libexec</b><br />In fact, you have to set <b>GOPATH</b> to the directory that contains the <b>src</b> directory that contains the source of go main packages.<br /><br />Set <b>GOPATH</b> to whatever you want that is different from <b>GOROOT.</b><br /><b><br /></b><div class="separator" style="clear: both; text-align: center;"><a href="http://3.bp.blogspot.com/-FP7oES_y-Ss/UpRfZ4UstmI/AAAAAAAAnvw/RqzDzulRgo4/s1600/go_gopher_color_logo_250x249%5B1%5D.png" imageanchor="1" style="margin-left: 1em; margin-right: 1em;"><img border="0" src="http://3.bp.blogspot.com/-FP7oES_y-Ss/UpRfZ4UstmI/AAAAAAAAnvw/RqzDzulRgo4/s1600/go_gopher_color_logo_250x249%5B1%5D.png" /></a></div><div style="text-align: center;"><b><br /></b></div><b><br /></b><b><br /></b><br /><br /></div>
