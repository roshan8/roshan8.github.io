
Recently we had a requirement to migrate repos from one github organization to another. Since we had 100+ projects, manual migration would have taken some considerable amount of time. So we decided to automate the repos migration with some good old shell scripts!

---

### Clone all the repo's in your current Github Org.
```sh
curl -sS -H "Authorization: token yourgithubtokenFromPresentOrg" https://api.github.com/orgs/yourPresentOrgName/repos?per_page=200 \
	| jq -r '.[].svn_url' \
	| while read svn_url; \
	do git clone "https://${svn_url##*://}"; done
``` 

### Create an empty projects on your new Github Org
```sh
for d in ./*; do
  curl -sS -H "Authorization: token yourgithubtokenFromNewOrg" -X POST https://api.github.com/orgs/yourNewOrgName/repos \
  -d  "{\"name\":\"${d/.\//}\",\"private\":true,\"has_issues\":true,\"has_wiki\":true,\"has_projects\":true}"
done;
``` 

### Change the remote git origin to new Org and push all the repos cloned from previous org
```sh
for d in ./*; do
  if [ -d "${d}" ]; then
    echo {$d};
    cd ${d};
    git remote set-url origin git@github.com:yourNewOrgName/${d/.\//}.git;
    git push;
    cd ..;
  fi
done
``` 

