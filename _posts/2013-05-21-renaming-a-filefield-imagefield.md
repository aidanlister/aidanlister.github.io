---
layout: post
title: Renaming a FileField / ImageField
section: developer
---
for trot in trots:
  if len(trot.photo.name):
    proper_path = get_trot_upload_path(trot, 'notrelevant.jpg')
    if trot.photo.name != proper_path:
      print trot.photo.name
      oldfile = trot.photo.file
      trot.photo.save(proper_path, trot.photo.file)
