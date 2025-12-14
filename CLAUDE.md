# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This repository maintains privacy policy pages for multiple App projects. Each policy file is a standalone static HTML webpage.

## Repository Structure

- Each app's privacy policy is stored as a separate HTML file
- Files should be self-contained static pages with embedded CSS and minimal JavaScript if needed
- Naming convention should clearly identify which app the policy belongs to (e.g., `app-name-privacy-policy.html`)

## Creating New Policy Pages

When creating a new privacy policy page:

1. **HTML Structure**: Use semantic HTML5 with proper document structure (`<!DOCTYPE html>`, `<html>`, `<head>`, `<body>`)
2. **Responsive Design**: Include viewport meta tag and ensure mobile-friendly layout
3. **Inline Styles**: Embed CSS within `<style>` tags in the `<head>` for portability
4. **Language**: Support multiple languages if needed, using appropriate `lang` attribute
5. **Accessibility**: Use proper heading hierarchy (`<h1>`, `<h2>`, etc.) and semantic elements

## Content Guidelines

Each privacy policy should typically include:

- App name and developer information
- Data collection practices
- Data usage and storage
- Third-party services
- User rights
- Contact information
- Last updated date

## Deployment

Since these are static HTML files, they can be:
- Hosted on any static web server
- Served via GitHub Pages
- Uploaded to CDN or cloud storage with public access
- Embedded in app webviews
