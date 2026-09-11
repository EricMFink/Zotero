# Better BibTex Citation Key Format

type(Case) + shorttitle(4,4).len.nopunct + Court.nopunct.prefix('_') + year; 
type(Statute) + CodeNumber.len.nopunct.prefix('_') + Code.nopunct + Section.nopunct; 
type(Statute) + shorttitle(3,3).len.nopunct + Code.nopunct.prefix('_') + Section.nopunct; 
type(Standard) + shorttitle(6,6).len.nopunct; 
type(NewspaperArticle) +  auth.len.nopunct + shorttitle(3,3).len.nopunct + PublicationTitle.prefix('_') + year.default(_nd); 
type(Document) +  auth.len.nopunct + shorttitle(3,3).len.nopunct +  year.default(_nd); year; 
type(WebPage) + auth.len.nopunct + shorttitle(4,4).len.nopunct + WebsiteTitle.len.nopunct.prefix('_') + year.default(_nd); 
type(BlogPost) + auth.nopunct + BlogTitle.len.nopunct.prefix('_') + year.default(_nd); 
auth.len.nopunct + year.default(_nd); 
shorttitle(3,3).len.nopunct + year.default(_nd)
