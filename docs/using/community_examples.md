# Community Usage Examples

This collects [github gists][github_gists] which HTCF users have written to 
serve as examples of HTCF usage patterns. We encourage you to write your own 
gists which describe common tasks you do on the cluster, and share them. This 
is a good way of keeping a record of how you use HTCF for yourself and future 
lab mates, and can help the whole HTCF community improve and optimize usage. 
Feel free to ask questions of the gist writer through the github interface.

- [Turning Exploratory Data Analysis into containerized cmd line scripts in R][eda_to_containerized_cmd]

	> This gist describes a method of interactively exploring data in R in a way that 
allows the user to simultaneously develop a command line script. Critically, this 
describes how to use [renv][renv_link] to manage a virtual environment. 
This makes it simple to create a [docker][docker_link] container. Since 
[singularity][singularity_link] can run docker containers, this means that you 
can easily turn EDA into reproducible results which may be easily run on HTCF.

[github_gists]: https://docs.github.com/en/get-started/writing-on-github/editing-and-sharing-content-with-gists/creating-gists
[renv_link]: https://rstudio.github.io/renv/articles/renv.html
[docker_link]: https://www.docker.com/
[singularity_link]: https://sylabs.io/singularity/
[eda_to_containerized_cmd]: https://gist.github.com/cmatKhan/e08a262b45bed5465301530b7319e6b3
